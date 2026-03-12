---
name: startup-gtm-monetization
description: >
  Activates elite startup go-to-market and monetization engineering capabilities. Use this skill
  for ANY task involving: Stripe integration (payments, subscriptions, webhooks, billing portal);
  RevenueCat for mobile in-app purchases (iOS App Store, Google Play); pricing page design and
  psychology; freemium-to-paid conversion flows; subscription management (upgrades, downgrades,
  cancellation flows); usage-based billing; trial periods; discount codes and coupons; affiliate
  systems; customer portal; invoice generation; dunning management (failed payment recovery);
  feature gating by plan; or any task that connects product features to revenue. Trigger when the
  user says "payments", "Stripe", "subscription", "pricing", "billing", "RevenueCat", "in-app
  purchase", "paywall", "upgrade flow", "trial", or "monetize". Ship revenue code correctly
  the first time — payment bugs are customer trust bugs.
---

# Elite Startup GTM & Monetization Engineer

You architect revenue systems that convert, retain, and recover — from the first free user
to enterprise contracts. Payments code must be correct, idempotent, and auditable.

---

## PART 1 — STRIPE INTEGRATION

### 1.1 Stripe Setup (Cloudflare Workers)

```typescript
// lib/stripe.ts
import Stripe from 'stripe';

export function getStripe(env: Env): Stripe {
  return new Stripe(env.STRIPE_SECRET_KEY, {
    apiVersion: '2024-12-18.acacia',
    httpClient: Stripe.createFetchHttpClient(),  // Required for Workers/Edge
  });
}

// Types
interface SubscriptionTier {
  id: 'free' | 'pro' | 'enterprise';
  name: string;
  stripePriceId: string;
  features: string[];
  limits: { projects: number; members: number; storage: number };
}

const TIERS: Record<string, SubscriptionTier> = {
  free: {
    id: 'free', name: 'Free',
    stripePriceId: '',
    features: ['3 projects', '1 member', '1GB storage'],
    limits: { projects: 3, members: 1, storage: 1 },
  },
  pro: {
    id: 'pro', name: 'Pro',
    stripePriceId: process.env.STRIPE_PRO_PRICE_ID!,
    features: ['Unlimited projects', '10 members', '100GB storage', 'Priority support'],
    limits: { projects: -1, members: 10, storage: 100 },
  },
};
```

### 1.2 Checkout Session Creation

```typescript
// POST /api/billing/checkout
export async function createCheckoutSession(
  workspaceId: string,
  priceId: string,
  env: Env
): Promise<string> {
  const stripe = getStripe(env);

  const workspace = await getWorkspace(workspaceId, env);
  let customerId = workspace.stripeCustomerId;

  // Create Stripe customer if first time
  if (!customerId) {
    const customer = await stripe.customers.create({
      email: workspace.ownerEmail,
      name: workspace.name,
      metadata: { workspaceId },
    });
    customerId = customer.id;
    await updateWorkspace(workspaceId, { stripeCustomerId: customerId }, env);
  }

  const session = await stripe.checkout.sessions.create({
    customer: customerId,
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    subscription_data: {
      metadata: { workspaceId },
      trial_period_days: 14,    // 14-day free trial
    },
    success_url: `${env.APP_URL}/settings/billing?session_id={CHECKOUT_SESSION_ID}&success=true`,
    cancel_url: `${env.APP_URL}/settings/billing?canceled=true`,
    allow_promotion_codes: true,
    automatic_tax: { enabled: true },
    customer_update: {
      address: 'auto',
      name: 'auto',
    },
  });

  return session.url!;
}
```

### 1.3 Webhook Handler (Critical — Must Be Idempotent)

```typescript
// POST /api/webhooks/stripe
export async function handleStripeWebhook(request: Request, env: Env): Promise<Response> {
  const stripe = getStripe(env);
  const body = await request.text();
  const sig = request.headers.get('stripe-signature');

  if (!sig) return new Response('No signature', { status: 400 });

  let event: Stripe.Event;
  try {
    event = await stripe.webhooks.constructEventAsync(body, sig, env.STRIPE_WEBHOOK_SECRET);
  } catch {
    return new Response('Invalid signature', { status: 400 });
  }

  // Idempotency: check if we've already processed this event
  const existing = await env.DB
    .prepare('SELECT id FROM subscription_events WHERE stripe_event_id = ?')
    .bind(event.id)
    .first();
  if (existing) return new Response('Already processed', { status: 200 });

  // Process event
  try {
    await processStripeEvent(event, env);

    // Record processed event
    await env.DB
      .prepare('INSERT INTO subscription_events (id, stripe_event_id, event_type, payload, processed_at) VALUES (?, ?, ?, ?, ?)')
      .bind(crypto.randomUUID(), event.id, event.type, JSON.stringify(event), new Date().toISOString())
      .run();

    return new Response('OK', { status: 200 });
  } catch (error) {
    console.error('Webhook processing error:', error);
    // Return 500 so Stripe retries
    return new Response('Processing error', { status: 500 });
  }
}

async function processStripeEvent(event: Stripe.Event, env: Env): Promise<void> {
  switch (event.type) {
    case 'checkout.session.completed': {
      const session = event.data.object as Stripe.CheckoutSession;
      if (session.mode !== 'subscription') break;

      const workspaceId = session.metadata?.workspaceId;
      if (!workspaceId) break;

      await activateSubscription(workspaceId, session.subscription as string, env);
      break;
    }

    case 'customer.subscription.updated': {
      const sub = event.data.object as Stripe.Subscription;
      const workspaceId = sub.metadata?.workspaceId;
      if (!workspaceId) break;

      await syncSubscription(workspaceId, sub, env);
      break;
    }

    case 'customer.subscription.deleted': {
      const sub = event.data.object as Stripe.Subscription;
      const workspaceId = sub.metadata?.workspaceId;
      if (!workspaceId) break;

      await downgradeToFree(workspaceId, env);
      break;
    }

    case 'invoice.payment_failed': {
      const invoice = event.data.object as Stripe.Invoice;
      await handlePaymentFailure(invoice, env);
      break;
    }

    case 'invoice.paid': {
      const invoice = event.data.object as Stripe.Invoice;
      await handlePaymentSuccess(invoice, env);
      break;
    }
  }
}
```

### 1.4 Customer Portal (Self-Service Billing)

```typescript
// POST /api/billing/portal
export async function createBillingPortalSession(
  workspaceId: string,
  env: Env
): Promise<string> {
  const stripe = getStripe(env);
  const workspace = await getWorkspace(workspaceId, env);

  if (!workspace.stripeCustomerId) {
    throw new Error('No billing account found');
  }

  const session = await stripe.billingPortal.sessions.create({
    customer: workspace.stripeCustomerId,
    return_url: `${env.APP_URL}/settings/billing`,
  });

  return session.url;
}
```

---

## PART 2 — FEATURE GATING

### 2.1 Plan Enforcement Middleware

```typescript
// Enforce plan limits before expensive operations
export async function enforcePlanLimit(
  workspaceId: string,
  resource: 'projects' | 'members' | 'api_calls',
  env: Env
): Promise<void> {
  const [subscription, currentUsage] = await Promise.all([
    getSubscription(workspaceId, env),
    getCurrentUsage(workspaceId, resource, env),
  ]);

  const tier = TIERS[subscription.plan];
  const limit = tier.limits[resource as keyof typeof tier.limits];

  if (limit !== -1 && currentUsage >= limit) {
    throw new PlanLimitError({
      resource,
      current: currentUsage,
      limit,
      plan: subscription.plan,
      upgradeUrl: `/settings/billing?upgrade=true&reason=${resource}`,
    });
  }
}

// Usage in route handler
app.post('/api/projects', async (c) => {
  const workspaceId = c.get('workspaceId');

  try {
    await enforcePlanLimit(workspaceId, 'projects', c.env);
  } catch (error) {
    if (error instanceof PlanLimitError) {
      return c.json({
        error: 'PLAN_LIMIT_EXCEEDED',
        message: `You've reached the ${error.limit} project limit on your ${error.plan} plan.`,
        upgradeUrl: error.upgradeUrl,
      }, 402);
    }
    throw error;
  }

  // ... create project
});
```

### 2.2 Feature Flag by Plan (React)

```tsx
function FeatureGate({
  feature,
  children,
  fallback,
}: {
  feature: keyof PlanFeatures;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}) {
  const { subscription } = useSubscription();
  const hasAccess = PLAN_FEATURES[subscription.plan]?.[feature] ?? false;

  if (!hasAccess) {
    return fallback ? <>{fallback}</> : (
      <div className="relative group">
        <div className="pointer-events-none opacity-40 select-none">
          {children}
        </div>
        <UpgradePrompt feature={feature} />
      </div>
    );
  }

  return <>{children}</>;
}

// Usage
<FeatureGate
  feature="advancedAnalytics"
  fallback={<UpgradeBanner feature="Advanced Analytics" />}
>
  <AnalyticsDashboard />
</FeatureGate>
```

---

## PART 3 — PRICING PAGE PSYCHOLOGY

### 3.1 Pricing Page Best Practices

```
Psychological levers that increase conversion:
  ✓ 3-tier structure: Free / Pro / Enterprise
    (Free anchors value; Enterprise makes Pro look reasonable)
  ✓ Highlight the recommended plan with visual emphasis
  ✓ Annual billing default with "Save X%" badge (typically 2 months free)
  ✓ List features as outcomes, not technical specs
    "Ship 10x faster" not "Unlimited API calls"
  ✓ Social proof below pricing (logos, testimonial, user count)
  ✓ FAQ section addressing top objections inline
  ✓ Money-back guarantee removes risk perception
  ✓ Show total annual price as monthly equivalent

What kills conversion:
  ✗ More than 4 tiers (choice paralysis)
  ✗ Feature matrix with 40 rows (cognitive overload)
  ✗ Hiding price (forces contact, loses self-serve)
  ✗ No free trial or free tier (too much risk for users)
  ✗ "Contact sales" without a price anchor for Enterprise
```

### 3.2 Pricing Toggle Component

```tsx
function PricingPage() {
  const [isAnnual, setIsAnnual] = useState(true);
  const annualDiscount = 0.2;  // 20% off

  const tiers = [
    {
      name: 'Starter',
      monthlyPrice: 0,
      description: 'For individuals exploring the product',
      features: ['3 projects', '1 team member', 'Community support'],
      cta: 'Start for free',
      ctaVariant: 'secondary' as const,
    },
    {
      name: 'Pro',
      monthlyPrice: 29,
      description: 'For growing teams',
      features: ['Unlimited projects', '10 team members', 'Priority support', 'Advanced analytics', 'Custom domains'],
      cta: 'Start 14-day trial',
      ctaVariant: 'primary' as const,
      recommended: true,
    },
    {
      name: 'Enterprise',
      monthlyPrice: 99,
      description: 'For organizations at scale',
      features: ['Everything in Pro', 'Unlimited members', 'SSO / SAML', 'SLA guarantee', 'Dedicated support'],
      cta: 'Talk to sales',
      ctaVariant: 'secondary' as const,
    },
  ];

  const displayPrice = (monthly: number) => {
    if (monthly === 0) return '$0';
    const price = isAnnual ? monthly * (1 - annualDiscount) : monthly;
    return `$${price.toFixed(0)}`;
  };

  return (
    <section className="py-24">
      <div className="text-center mb-16">
        <h2 className="text-4xl font-bold mb-4">Simple, transparent pricing</h2>
        <p className="text-text-secondary text-lg mb-8">No hidden fees. Cancel anytime.</p>

        {/* Billing toggle */}
        <div className="inline-flex items-center gap-3 bg-bg-subtle rounded-full p-1">
          <button
            onClick={() => setIsAnnual(false)}
            className={cn('px-4 py-2 rounded-full text-sm font-medium transition-all',
              !isAnnual ? 'bg-surface shadow text-text-primary' : 'text-text-secondary')}
          >
            Monthly
          </button>
          <button
            onClick={() => setIsAnnual(true)}
            className={cn('px-4 py-2 rounded-full text-sm font-medium transition-all flex items-center gap-2',
              isAnnual ? 'bg-surface shadow text-text-primary' : 'text-text-secondary')}
          >
            Annual
            <span className="bg-green-100 text-green-700 text-xs px-1.5 py-0.5 rounded-full">
              Save 20%
            </span>
          </button>
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6 max-w-5xl mx-auto">
        {tiers.map(tier => (
          <div
            key={tier.name}
            className={cn(
              'rounded-2xl p-8 border',
              tier.recommended
                ? 'border-brand bg-brand-subtle ring-2 ring-brand scale-105'
                : 'border-border bg-surface'
            )}
          >
            {tier.recommended && (
              <div className="text-xs font-semibold text-brand uppercase tracking-wide mb-4">
                Most Popular
              </div>
            )}
            <h3 className="text-xl font-semibold mb-2">{tier.name}</h3>
            <p className="text-text-secondary text-sm mb-6">{tier.description}</p>

            <div className="mb-6">
              <span className="text-4xl font-bold">{displayPrice(tier.monthlyPrice)}</span>
              {tier.monthlyPrice > 0 && (
                <span className="text-text-secondary">/mo{isAnnual ? ', billed annually' : ''}</span>
              )}
            </div>

            <Button variant={tier.ctaVariant} className="w-full mb-8">
              {tier.cta}
            </Button>

            <ul className="space-y-3">
              {tier.features.map(feature => (
                <li key={feature} className="flex items-center gap-2 text-sm">
                  <Check className="h-4 w-4 text-success shrink-0" />
                  {feature}
                </li>
              ))}
            </ul>
          </div>
        ))}
      </div>
    </section>
  );
}
```

---

## PART 4 — REVENUECAT (MOBILE IAP)

### 4.1 React Native Setup

```typescript
// services/purchases.ts
import Purchases, { LOG_LEVEL, PurchasesPackage } from 'react-native-purchases';
import { Platform } from 'react-native';

export async function initPurchases(userId: string): Promise<void> {
  Purchases.setLogLevel(LOG_LEVEL.VERBOSE);

  const apiKey = Platform.select({
    ios:     process.env.EXPO_PUBLIC_RC_IOS_KEY!,
    android: process.env.EXPO_PUBLIC_RC_ANDROID_KEY!,
  })!;

  await Purchases.configure({ apiKey, appUserID: userId });
}

export async function getOfferings() {
  const offerings = await Purchases.getOfferings();
  return offerings.current;
}

export async function purchasePackage(pkg: PurchasesPackage) {
  try {
    const { customerInfo } = await Purchases.purchasePackage(pkg);
    return { success: true, customerInfo };
  } catch (error: any) {
    if (!error.userCancelled) throw error;
    return { success: false, userCancelled: true };
  }
}

export async function restorePurchases() {
  const customerInfo = await Purchases.restorePurchases();
  return customerInfo;
}

export function hasActiveSubscription(customerInfo: CustomerInfo): boolean {
  return Object.keys(customerInfo.entitlements.active).length > 0;
}

export function getActiveTier(customerInfo: CustomerInfo): 'free' | 'pro' | 'enterprise' {
  const active = customerInfo.entitlements.active;
  if (active['enterprise']) return 'enterprise';
  if (active['pro']) return 'pro';
  return 'free';
}
```

### 4.2 Paywall Component (React Native)

```tsx
function PaywallScreen({ onDismiss }: { onDismiss: () => void }) {
  const [offerings, setOfferings] = useState<PurchasesOffering | null>(null);
  const [selectedPackage, setSelectedPackage] = useState<PurchasesPackage | null>(null);
  const [isPurchasing, setIsPurchasing] = useState(false);

  useEffect(() => {
    getOfferings().then(setOfferings);
  }, []);

  const handlePurchase = async () => {
    if (!selectedPackage) return;
    setIsPurchasing(true);
    try {
      const result = await purchasePackage(selectedPackage);
      if (result.success) {
        onDismiss();
        showSuccessToast('Welcome to Pro! 🎉');
      }
    } catch (error) {
      showErrorToast('Purchase failed. Please try again.');
    } finally {
      setIsPurchasing(false);
    }
  };

  return (
    <View style={styles.container}>
      {/* Hero section */}
      <View style={styles.hero}>
        <Text style={styles.title}>Unlock Pro Features</Text>
        <Text style={styles.subtitle}>Join 10,000+ users growing with Pro</Text>
      </View>

      {/* Feature highlights */}
      {PRO_FEATURES.map(feature => (
        <View key={feature.title} style={styles.featureRow}>
          <View style={[styles.featureIcon, { backgroundColor: feature.color + '20' }]}>
            <feature.Icon size={20} color={feature.color} />
          </View>
          <View>
            <Text style={styles.featureTitle}>{feature.title}</Text>
            <Text style={styles.featureDesc}>{feature.description}</Text>
          </View>
        </View>
      ))}

      {/* Package selection */}
      {offerings?.availablePackages.map(pkg => (
        <TouchableOpacity
          key={pkg.identifier}
          style={[styles.packageOption, selectedPackage === pkg && styles.selectedPackage]}
          onPress={() => setSelectedPackage(pkg)}
        >
          <Text>{pkg.packageType === 'ANNUAL' ? 'Annual' : 'Monthly'}</Text>
          <Text style={styles.price}>{pkg.product.priceString}</Text>
          {pkg.packageType === 'ANNUAL' && (
            <View style={styles.saveBadge}>
              <Text style={styles.saveText}>Save 40%</Text>
            </View>
          )}
        </TouchableOpacity>
      ))}

      <Button onPress={handlePurchase} isLoading={isPurchasing}>
        Start Free Trial
      </Button>

      <TouchableOpacity onPress={() => restorePurchases()}>
        <Text style={styles.restoreLink}>Restore purchases</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## PART 5 — DUNNING & RETENTION

### 5.1 Failed Payment Recovery

```typescript
// Triggered by invoice.payment_failed webhook
async function handlePaymentFailure(invoice: Stripe.Invoice, env: Env): Promise<void> {
  const workspaceId = await getWorkspaceByCustomer(invoice.customer as string, env);
  const attemptCount = invoice.attempt_count;

  // Graduated response based on attempt number
  if (attemptCount === 1) {
    await sendEmail({
      to: invoice.customer_email!,
      template: 'payment_failed_first',
      data: { updateUrl: await createBillingPortalUrl(invoice.customer as string, env) },
    });
  } else if (attemptCount === 2) {
    await sendEmail({
      to: invoice.customer_email!,
      template: 'payment_failed_urgent',
      data: { daysRemaining: 3 },
    });
  } else if (attemptCount >= 3) {
    // Downgrade but keep data for 30 days
    await downgradeToFree(workspaceId, env);
    await scheduleDataDeletion(workspaceId, 30, env);  // 30-day grace period
    await sendEmail({
      to: invoice.customer_email!,
      template: 'subscription_paused',
    });
  }
}
```

### 5.2 Cancellation Flow (Save the Subscription)

```tsx
function CancellationFlow({ onConfirm, onCancel }: Props) {
  const [step, setStep] = useState<'reason' | 'offer' | 'confirm'>('reason');
  const [reason, setReason] = useState('');

  const REASONS = [
    { id: 'too_expensive', label: 'Too expensive', offer: 'discount' },
    { id: 'not_using', label: "Not using it enough", offer: 'pause' },
    { id: 'missing_feature', label: 'Missing a feature I need', offer: 'feedback' },
    { id: 'switching', label: 'Switching to another product', offer: null },
    { id: 'other', label: 'Other', offer: null },
  ];

  const selectedReason = REASONS.find(r => r.id === reason);

  return (
    <Modal>
      {step === 'reason' && (
        <div>
          <h2>We're sorry to see you go</h2>
          <p>Help us improve by telling us why you're canceling</p>
          {REASONS.map(r => (
            <button key={r.id} onClick={() => { setReason(r.id); setStep(r.offer ? 'offer' : 'confirm'); }}>
              {r.label}
            </button>
          ))}
        </div>
      )}

      {step === 'offer' && selectedReason?.offer === 'discount' && (
        <div>
          <h2>How about 50% off for 3 months?</h2>
          <Button onClick={() => applyDiscount('50_off_3months')}>Apply Discount</Button>
          <button onClick={() => setStep('confirm')}>No thanks, cancel anyway</button>
        </div>
      )}

      {step === 'confirm' && (
        <div>
          <h2>Are you sure?</h2>
          <p>You'll lose access to Pro features at end of billing period.</p>
          <Button variant="danger" onClick={onConfirm}>Yes, cancel subscription</Button>
          <Button variant="secondary" onClick={onCancel}>Keep my subscription</Button>
        </div>
      )}
    </Modal>
  );
}
```

---

## PART 6 — MONETIZATION CHECKLIST

```
Technical:
□ Stripe webhook endpoint verified and idempotent
□ Stripe CLI used for local webhook testing (stripe listen)
□ All plan limits enforced server-side (not just UI)
□ Subscription status synced on every authenticated request
□ Grace period for failed payments (don't cut off immediately)
□ Billing portal available for self-service management
□ Annual + monthly pricing options
□ Trial period implemented (14 days recommended)

UX:
□ Upgrade prompts contextual (at the moment limit is hit)
□ Pricing page A/B tested (use PostHog or similar)
□ Cancellation flow has at least one save attempt
□ Reactivation flow smooth (no data lost during grace period)
□ Invoice emails professional and branded
□ Clear receipt of payment confirmation
```

---

> **Core Principle**: Revenue is the feature that funds all other features.
> Build payments code with the same rigor as security code — idempotent,
> auditable, and always human-friendly when things go wrong.
