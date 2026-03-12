---
name: design-system-creator
description: >
  Activates elite design system creation capabilities. Use this skill for ANY task involving:
  building a design system or component library from scratch; creating design tokens (colors,
  typography, spacing, shadows, radii); building reusable React component libraries; Tailwind CSS
  configuration and custom themes; CSS custom property systems; Storybook setup and stories;
  Figma token exports; accessibility-compliant component APIs; dark/light mode systems; branded
  UI kits; icon systems; animation tokens; or any task where the goal is a systematic, scalable
  approach to UI rather than one-off components. Trigger when the user says "design system",
  "component library", "tokens", "theme", "brand", "style guide", "Storybook", "consistent UI",
  or "reusable components". Never build ad-hoc — always build systematic.
---

# Elite Design System Creator

You architect and build production-grade design systems — the foundation that makes every UI
consistent, accessible, and fast to build. A great design system is invisible when it works
and painful when it's missing.

---

## PART 1 — DESIGN TOKEN ARCHITECTURE

### 1.1 Three-Tier Token System

```
Tier 1: Primitive tokens (raw values — brand-agnostic)
  color-blue-500: #3b82f6
  font-size-16: 1rem
  space-4: 16px

Tier 2: Semantic tokens (role-based — reference primitives)
  color-background-primary: {color-gray-950}
  color-text-primary: {color-gray-50}
  color-interactive-default: {color-blue-500}
  color-interactive-hover: {color-blue-400}

Tier 3: Component tokens (component-scoped — reference semantic)
  button-primary-background: {color-interactive-default}
  button-primary-background-hover: {color-interactive-hover}
  button-primary-text: {color-white}
```

### 1.2 Full Token System (CSS Custom Properties)

```css
/* tokens.css */
:root {
  /* ─── Primitive: Colors ────────────────────────────── */
  --primitive-gray-0:    #ffffff;
  --primitive-gray-50:   #fafafa;
  --primitive-gray-100:  #f4f4f5;
  --primitive-gray-200:  #e4e4e7;
  --primitive-gray-300:  #d4d4d8;
  --primitive-gray-400:  #a1a1aa;
  --primitive-gray-500:  #71717a;
  --primitive-gray-600:  #52525b;
  --primitive-gray-700:  #3f3f46;
  --primitive-gray-800:  #27272a;
  --primitive-gray-900:  #18181b;
  --primitive-gray-950:  #09090b;

  --primitive-brand-50:  #eff6ff;
  --primitive-brand-100: #dbeafe;
  --primitive-brand-200: #bfdbfe;
  --primitive-brand-300: #93c5fd;
  --primitive-brand-400: #60a5fa;
  --primitive-brand-500: #3b82f6;
  --primitive-brand-600: #2563eb;
  --primitive-brand-700: #1d4ed8;
  --primitive-brand-800: #1e40af;
  --primitive-brand-900: #1e3a8a;

  --primitive-success-500: #22c55e;
  --primitive-warning-500: #f59e0b;
  --primitive-error-500:   #ef4444;
  --primitive-info-500:    #3b82f6;

  /* ─── Primitive: Typography ────────────────────────── */
  --primitive-font-display: 'Clash Display', 'Cabinet Grotesk', sans-serif;
  --primitive-font-body:    'DM Sans', 'Outfit', sans-serif;
  --primitive-font-mono:    'JetBrains Mono', 'Geist Mono', monospace;

  --primitive-size-xs:   0.75rem;    /* 12px */
  --primitive-size-sm:   0.875rem;   /* 14px */
  --primitive-size-base: 1rem;       /* 16px */
  --primitive-size-lg:   1.125rem;   /* 18px */
  --primitive-size-xl:   1.25rem;    /* 20px */
  --primitive-size-2xl:  1.5rem;     /* 24px */
  --primitive-size-3xl:  1.875rem;   /* 30px */
  --primitive-size-4xl:  2.25rem;    /* 36px */
  --primitive-size-5xl:  3rem;       /* 48px */
  --primitive-size-6xl:  3.75rem;    /* 60px */

  /* ─── Primitive: Spacing ────────────────────────────── */
  --primitive-space-0:   0;
  --primitive-space-1:   0.25rem;    /* 4px */
  --primitive-space-2:   0.5rem;     /* 8px */
  --primitive-space-3:   0.75rem;    /* 12px */
  --primitive-space-4:   1rem;       /* 16px */
  --primitive-space-5:   1.25rem;    /* 20px */
  --primitive-space-6:   1.5rem;     /* 24px */
  --primitive-space-8:   2rem;       /* 32px */
  --primitive-space-10:  2.5rem;     /* 40px */
  --primitive-space-12:  3rem;       /* 48px */
  --primitive-space-16:  4rem;       /* 64px */
  --primitive-space-20:  5rem;       /* 80px */
  --primitive-space-24:  6rem;       /* 96px */
  --primitive-space-32:  8rem;       /* 128px */

  /* ─── Primitive: Radii ─────────────────────────────── */
  --primitive-radius-none: 0;
  --primitive-radius-sm:   0.25rem;  /* 4px */
  --primitive-radius-md:   0.5rem;   /* 8px */
  --primitive-radius-lg:   0.75rem;  /* 12px */
  --primitive-radius-xl:   1rem;     /* 16px */
  --primitive-radius-2xl:  1.5rem;   /* 24px */
  --primitive-radius-full: 9999px;

  /* ─── Semantic Tokens (Light Mode Default) ─────────── */
  --color-bg:           var(--primitive-gray-0);
  --color-bg-subtle:    var(--primitive-gray-50);
  --color-surface:      var(--primitive-gray-0);
  --color-surface-raised: var(--primitive-gray-50);
  --color-border:       var(--primitive-gray-200);
  --color-border-strong: var(--primitive-gray-300);

  --color-text-primary:   var(--primitive-gray-950);
  --color-text-secondary: var(--primitive-gray-600);
  --color-text-disabled:  var(--primitive-gray-400);
  --color-text-inverse:   var(--primitive-gray-0);

  --color-brand:          var(--primitive-brand-500);
  --color-brand-hover:    var(--primitive-brand-600);
  --color-brand-active:   var(--primitive-brand-700);
  --color-brand-subtle:   var(--primitive-brand-50);
  --color-brand-text:     var(--primitive-brand-700);

  --color-success:        var(--primitive-success-500);
  --color-warning:        var(--primitive-warning-500);
  --color-error:          var(--primitive-error-500);
  --color-info:           var(--primitive-info-500);

  --shadow-sm:   0 1px 2px rgba(0,0,0,0.06), 0 1px 3px rgba(0,0,0,0.1);
  --shadow-md:   0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06);
  --shadow-lg:   0 10px 15px rgba(0,0,0,0.1), 0 4px 6px rgba(0,0,0,0.05);
  --shadow-xl:   0 20px 25px rgba(0,0,0,0.1), 0 10px 10px rgba(0,0,0,0.04);

  /* ─── Animation ────────────────────────────────────── */
  --duration-fast:    100ms;
  --duration-normal:  200ms;
  --duration-slow:    300ms;
  --duration-slower:  500ms;
  --ease-default:     cubic-bezier(0.4, 0, 0.2, 1);
  --ease-in:          cubic-bezier(0.4, 0, 1, 1);
  --ease-out:         cubic-bezier(0, 0, 0.2, 1);
  --ease-spring:      cubic-bezier(0.16, 1, 0.3, 1);
}

/* Dark Mode */
[data-theme="dark"] {
  --color-bg:           var(--primitive-gray-950);
  --color-bg-subtle:    var(--primitive-gray-900);
  --color-surface:      var(--primitive-gray-900);
  --color-surface-raised: var(--primitive-gray-800);
  --color-border:       var(--primitive-gray-800);
  --color-border-strong: var(--primitive-gray-700);

  --color-text-primary:   var(--primitive-gray-50);
  --color-text-secondary: var(--primitive-gray-400);
  --color-text-disabled:  var(--primitive-gray-600);
  --color-text-inverse:   var(--primitive-gray-950);

  --color-brand-subtle:   rgba(59, 130, 246, 0.15);
  --color-brand-text:     var(--primitive-brand-400);

  --shadow-sm:   0 1px 2px rgba(0,0,0,0.4);
  --shadow-md:   0 4px 6px rgba(0,0,0,0.4);
  --shadow-lg:   0 10px 15px rgba(0,0,0,0.5);
  --shadow-xl:   0 20px 25px rgba(0,0,0,0.6);
}
```

---

## PART 2 — COMPONENT LIBRARY (REACT + TYPESCRIPT)

### 2.1 Button Component

```tsx
// components/ui/Button.tsx
import { forwardRef, ButtonHTMLAttributes } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';
import { Loader2 } from 'lucide-react';

const buttonVariants = cva(
  // Base styles
  [
    'inline-flex items-center justify-center gap-2 rounded-lg font-medium',
    'transition-all duration-200 ease-out',
    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-brand focus-visible:ring-offset-2',
    'disabled:pointer-events-none disabled:opacity-50',
    'select-none',
  ],
  {
    variants: {
      variant: {
        primary:   'bg-brand text-white hover:bg-brand-hover active:bg-brand-active shadow-sm',
        secondary: 'bg-surface border border-border text-text-primary hover:bg-bg-subtle shadow-sm',
        ghost:     'text-text-primary hover:bg-bg-subtle',
        danger:    'bg-error text-white hover:bg-red-600 active:bg-red-700 shadow-sm',
        link:      'text-brand underline-offset-4 hover:underline p-0 h-auto',
      },
      size: {
        sm:   'h-8  px-3  text-sm',
        md:   'h-10 px-4  text-sm',
        lg:   'h-11 px-6  text-base',
        xl:   'h-12 px-8  text-base',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  }
);

interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, isLoading, leftIcon, rightIcon, children, disabled, ...props }, ref) => (
    <button
      ref={ref}
      className={cn(buttonVariants({ variant, size }), className)}
      disabled={disabled || isLoading}
      aria-busy={isLoading}
      {...props}
    >
      {isLoading ? (
        <Loader2 className="h-4 w-4 animate-spin" aria-hidden />
      ) : leftIcon ? (
        <span className="shrink-0" aria-hidden>{leftIcon}</span>
      ) : null}
      {children}
      {!isLoading && rightIcon && (
        <span className="shrink-0" aria-hidden>{rightIcon}</span>
      )}
    </button>
  )
);
Button.displayName = 'Button';
```

### 2.2 Input Component

```tsx
// components/ui/Input.tsx
import { forwardRef, InputHTMLAttributes, useId } from 'react';
import { cn } from '@/lib/utils';

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  helperText?: string;
  error?: string;
  leftAddon?: React.ReactNode;
  rightAddon?: React.ReactNode;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ className, label, helperText, error, leftAddon, rightAddon, id: externalId, ...props }, ref) => {
    const generatedId = useId();
    const id = externalId ?? generatedId;
    const errorId = `${id}-error`;
    const helperId = `${id}-helper`;

    return (
      <div className="flex flex-col gap-1.5">
        {label && (
          <label htmlFor={id} className="text-sm font-medium text-text-primary">
            {label}
            {props.required && <span className="text-error ml-1" aria-label="required">*</span>}
          </label>
        )}

        <div className="relative flex items-center">
          {leftAddon && (
            <div className="absolute left-3 flex items-center text-text-secondary" aria-hidden>
              {leftAddon}
            </div>
          )}

          <input
            ref={ref}
            id={id}
            aria-invalid={!!error}
            aria-describedby={cn(error && errorId, helperText && helperId)}
            className={cn(
              'w-full rounded-lg border bg-surface px-3 py-2.5',
              'text-sm text-text-primary placeholder:text-text-secondary',
              'transition-colors duration-150',
              'focus:outline-none focus:ring-2 focus:ring-brand focus:border-brand',
              'disabled:cursor-not-allowed disabled:opacity-50',
              error
                ? 'border-error focus:ring-error'
                : 'border-border hover:border-border-strong',
              leftAddon && 'pl-10',
              rightAddon && 'pr-10',
              className
            )}
            {...props}
          />

          {rightAddon && (
            <div className="absolute right-3 flex items-center text-text-secondary" aria-hidden>
              {rightAddon}
            </div>
          )}
        </div>

        {error && (
          <p id={errorId} role="alert" className="text-xs text-error flex items-center gap-1">
            <AlertCircle className="h-3 w-3 shrink-0" aria-hidden />
            {error}
          </p>
        )}
        {helperText && !error && (
          <p id={helperId} className="text-xs text-text-secondary">{helperText}</p>
        )}
      </div>
    );
  }
);
Input.displayName = 'Input';
```

### 2.3 Card Component

```tsx
// components/ui/Card.tsx
import { cn } from '@/lib/utils';

interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  as?: React.ElementType;
  variant?: 'default' | 'ghost' | 'outlined';
  padding?: 'none' | 'sm' | 'md' | 'lg';
  hoverable?: boolean;
}

export function Card({
  as: Component = 'div',
  className,
  variant = 'default',
  padding = 'md',
  hoverable,
  children,
  ...props
}: CardProps) {
  return (
    <Component
      className={cn(
        'rounded-xl',
        {
          'bg-surface border border-border shadow-sm': variant === 'default',
          'bg-transparent': variant === 'ghost',
          'bg-transparent border border-border': variant === 'outlined',
        },
        {
          'p-0':  padding === 'none',
          'p-4':  padding === 'sm',
          'p-6':  padding === 'md',
          'p-8':  padding === 'lg',
        },
        hoverable && 'transition-all duration-200 hover:-translate-y-0.5 hover:shadow-md cursor-pointer',
        className
      )}
      {...props}
    >
      {children}
    </Component>
  );
}

Card.Header = function CardHeader({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('mb-4', className)} {...props} />;
};

Card.Title = function CardTitle({ className, ...props }: React.HTMLAttributes<HTMLHeadingElement>) {
  return <h3 className={cn('text-lg font-semibold text-text-primary', className)} {...props} />;
};

Card.Content = function CardContent({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('text-sm text-text-secondary', className)} {...props} />;
};

Card.Footer = function CardFooter({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('mt-4 pt-4 border-t border-border flex items-center', className)} {...props} />;
};
```

---

## PART 3 — TAILWIND CONFIGURATION

### 3.1 tailwind.config.ts with Design Tokens

```typescript
import type { Config } from 'tailwindcss';

export default {
  content: ['./src/**/*.{ts,tsx}'],
  darkMode: ['selector', '[data-theme="dark"]'],
  theme: {
    extend: {
      colors: {
        // Map CSS variables to Tailwind utilities
        brand: {
          DEFAULT: 'var(--color-brand)',
          hover:   'var(--color-brand-hover)',
          active:  'var(--color-brand-active)',
          subtle:  'var(--color-brand-subtle)',
          text:    'var(--color-brand-text)',
        },
        bg: {
          DEFAULT: 'var(--color-bg)',
          subtle:  'var(--color-bg-subtle)',
        },
        surface: {
          DEFAULT: 'var(--color-surface)',
          raised:  'var(--color-surface-raised)',
        },
        border: {
          DEFAULT: 'var(--color-border)',
          strong:  'var(--color-border-strong)',
        },
        text: {
          primary:   'var(--color-text-primary)',
          secondary: 'var(--color-text-secondary)',
          disabled:  'var(--color-text-disabled)',
          inverse:   'var(--color-text-inverse)',
        },
        error:   'var(--color-error)',
        success: 'var(--color-success)',
        warning: 'var(--color-warning)',
        info:    'var(--color-info)',
      },
      fontFamily: {
        display: 'var(--primitive-font-display)',
        body:    'var(--primitive-font-body)',
        mono:    'var(--primitive-font-mono)',
      },
      boxShadow: {
        sm: 'var(--shadow-sm)',
        md: 'var(--shadow-md)',
        lg: 'var(--shadow-lg)',
        xl: 'var(--shadow-xl)',
      },
      transitionTimingFunction: {
        spring: 'var(--ease-spring)',
      },
      borderRadius: {
        sm:   'var(--primitive-radius-sm)',
        md:   'var(--primitive-radius-md)',
        lg:   'var(--primitive-radius-lg)',
        xl:   'var(--primitive-radius-xl)',
        '2xl': 'var(--primitive-radius-2xl)',
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
} satisfies Config;
```

---

## PART 4 — STORYBOOK SETUP

### 4.1 Story Template

```typescript
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';
import { Mail, ArrowRight } from 'lucide-react';

const meta: Meta<typeof Button> = {
  title: 'UI/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'Primary interactive element. Use for user-initiated actions.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost', 'danger', 'link'],
    },
    size: { control: 'select', options: ['sm', 'md', 'lg', 'xl', 'icon'] },
    isLoading: { control: 'boolean' },
    disabled: { control: 'boolean' },
  },
};
export default meta;

type Story = StoryObj<typeof Button>;

export const Primary: Story = { args: { children: 'Button' } };
export const Secondary: Story = { args: { children: 'Button', variant: 'secondary' } };
export const Danger: Story = { args: { children: 'Delete', variant: 'danger' } };
export const Loading: Story = { args: { children: 'Saving...', isLoading: true } };
export const WithIcon: Story = {
  args: {
    children: 'Send email',
    leftIcon: <Mail className="h-4 w-4" />,
  },
};

export const AllVariants: Story = {
  render: () => (
    <div className="flex flex-wrap gap-3 items-center">
      {(['primary', 'secondary', 'ghost', 'danger'] as const).map(variant => (
        <Button key={variant} variant={variant}>{variant}</Button>
      ))}
    </div>
  ),
};
```

---

## PART 5 — ACCESSIBILITY CHECKLIST

```
Color & Contrast:
□ Text contrast ≥ 4.5:1 (body) / 3:1 (large text, icons)
□ Never use color as the only indicator (add icon/label)
□ Focus ring visible in both light and dark mode

Interactive Elements:
□ All buttons/links have accessible labels (aria-label if icon-only)
□ Form inputs linked to labels via htmlFor/id
□ Error messages use role="alert" and are in DOM (not just styled)
□ Disabled state communicates to screen readers

Keyboard Navigation:
□ All interactive elements reachable via Tab
□ Focus order matches visual order
□ No keyboard traps
□ Escape closes modals/popovers
□ Arrow keys navigate within menus/tabs

Motion:
□ prefers-reduced-motion respected — all animations conditional
□ No content that flashes > 3 times per second

Semantics:
□ Heading hierarchy: h1 → h2 → h3 (never skip levels)
□ Landmark regions: header, main, nav, footer
□ Lists use ul/ol, not divs with margin
□ Icons have aria-hidden="true" when decorative
```

---

> **Core Principle**: A design system is a product, not a task. It needs a roadmap,
> a changelog, versioning, and adoption metrics. The best design system is one that
> developers actually want to use — because it makes the right thing the easy thing.
