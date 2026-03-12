---
name: web-scraping-data-pipelines
description: >
  Activates elite web scraping and data pipeline engineering capabilities. Use this skill for
  ANY task involving: web scraping with Playwright, Puppeteer, or Cheerio; browser automation;
  structured data extraction from HTML/PDF/images; ETL pipelines (Extract, Transform, Load);
  data cleaning and normalization; scheduled data collection (Cloudflare Cron Workers); rate
  limiting and respectful scraping; proxy rotation; anti-bot bypass techniques; data storage
  pipelines into databases or object storage; news/content aggregation systems; price monitoring;
  change detection; or any workflow that collects data from external sources and transforms it
  for use. Trigger when the user says "scrape", "crawl", "extract data", "automate browser",
  "data pipeline", "ETL", "monitor prices", "collect data", "watch for changes", or "aggregate
  content". Always scrape respectfully and legally.
---

# Elite Web Scraping & Data Pipeline Engineer

You build data collection systems that are reliable, respectful, and resilient. You handle
anti-bot measures professionally, transform data cleanly, and build pipelines that don't
break when the target site changes.

---

## PART 1 — SCRAPING TECHNOLOGY SELECTION

### 1.1 Tool Selection Framework

```
Static HTML (no JavaScript required)?
→ Cheerio (fast, lightweight, Node.js jQuery-like)
  or JSDOM (full DOM parsing without browser)

JavaScript-rendered content, simple automation?
→ Playwright (preferred — more reliable, better API)
  or Puppeteer (older, Google-backed)

Need to appear as real user, handle complex anti-bot?
→ Playwright + stealth plugin + residential proxies

High-volume, lightweight scraping at edge?
→ Cloudflare Worker + fetch + Cheerio (no browser needed)

PDF extraction?
→ pdfjs-dist for text, Claude Vision API for scanned/complex PDFs

Structured data from LLM-parsed pages?
→ fetch HTML → Claude with extraction prompt → JSON
```

---

## PART 2 — PLAYWRIGHT SCRAPING

### 2.1 Production Playwright Setup

```typescript
import { chromium, Browser, Page } from 'playwright';

interface ScraperOptions {
  headless?: boolean;
  proxyUrl?: string;
  userAgent?: string;
  timeout?: number;
}

export class PlaywrightScraper {
  private browser: Browser | null = null;

  async init(options: ScraperOptions = {}): Promise<void> {
    this.browser = await chromium.launch({
      headless: options.headless ?? true,
      proxy: options.proxyUrl ? { server: options.proxyUrl } : undefined,
      args: [
        '--no-sandbox',
        '--disable-setuid-sandbox',
        '--disable-dev-shm-usage',
        '--disable-accelerated-2d-canvas',
        '--disable-gpu',
      ],
    });
  }

  async newPage(options: ScraperOptions = {}): Promise<Page> {
    if (!this.browser) throw new Error('Browser not initialized');

    const context = await this.browser.newContext({
      userAgent: options.userAgent ?? this.randomUserAgent(),
      viewport: { width: 1280 + Math.floor(Math.random() * 100), height: 800 },
      locale: 'en-US',
      timezoneId: 'America/New_York',
      // Block unnecessary resources to speed up scraping
      extraHTTPHeaders: {
        'Accept-Language': 'en-US,en;q=0.9',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
      },
    });

    // Block images, fonts, media to speed up (remove if needed for JS execution)
    await context.route('**/*.{png,jpg,jpeg,gif,webp,svg,woff,woff2,ttf,mp4,webm}', r => r.abort());

    const page = await context.newPage();
    page.setDefaultTimeout(options.timeout ?? 30_000);

    return page;
  }

  async scrapeWithRetry<T>(
    url: string,
    extractor: (page: Page) => Promise<T>,
    retries = 3
  ): Promise<T> {
    const page = await this.newPage();

    for (let attempt = 1; attempt <= retries; attempt++) {
      try {
        await page.goto(url, { waitUntil: 'networkidle', timeout: 30_000 });

        // Human-like delay
        await page.waitForTimeout(500 + Math.random() * 1000);

        return await extractor(page);
      } catch (error) {
        console.error(`Attempt ${attempt}/${retries} failed for ${url}:`, error);
        if (attempt === retries) throw error;

        // Exponential backoff
        await page.waitForTimeout(Math.pow(2, attempt) * 1000 + Math.random() * 1000);
      } finally {
        if (attempt === retries) await page.close();
      }
    }

    throw new Error('Should not reach here');
  }

  async close(): Promise<void> {
    await this.browser?.close();
    this.browser = null;
  }

  private randomUserAgent(): string {
    const agents = [
      'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36',
      'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36',
      'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36',
    ];
    return agents[Math.floor(Math.random() * agents.length)];
  }
}
```

### 2.2 Structured Data Extraction

```typescript
// Cheerio for fast static HTML parsing
import * as cheerio from 'cheerio';

export async function extractProductData(html: string): Promise<Product[]> {
  const $ = cheerio.load(html);
  const products: Product[] = [];

  $('.product-card').each((_, element) => {
    const el = $(element);

    const priceText = el.find('.price').text().trim();
    const priceCents = parsePriceCents(priceText);

    products.push({
      id: el.data('product-id') as string,
      title: el.find('.product-title').text().trim(),
      priceCents,
      rating: parseFloat(el.find('.rating').attr('data-rating') ?? '0'),
      imageUrl: el.find('img').attr('src') ?? null,
      url: el.find('a').attr('href') ?? null,
    });
  });

  return products.filter(p => p.title && p.priceCents > 0);
}

function parsePriceCents(priceText: string): number {
  // Handle: "$19.99", "19.99", "£19.99", "19,99 €"
  const cleaned = priceText.replace(/[^0-9.,]/g, '').replace(',', '.');
  const float = parseFloat(cleaned);
  return Math.round(float * 100);
}
```

### 2.3 AI-Powered Extraction (for complex/variable pages)

```typescript
import Anthropic from '@anthropic-ai/sdk';

export async function extractWithAI<T>(
  html: string,
  schema: string,       // JSON schema description
  instructions: string
): Promise<T> {
  const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

  // Truncate HTML to relevant parts (saves tokens)
  const cleanedHtml = cleanHtml(html);

  const response = await client.messages.create({
    model: 'claude-haiku-4-5-20251001',  // Fast + cheap for extraction
    max_tokens: 2048,
    messages: [{
      role: 'user',
      content: `Extract structured data from this HTML.

Instructions: ${instructions}

Return ONLY valid JSON matching this schema: ${schema}
No explanation, no markdown. Just the JSON object.

HTML:
${cleanedHtml}`,
    }],
  });

  const text = response.content[0].type === 'text' ? response.content[0].text : '';
  return JSON.parse(text);
}

function cleanHtml(html: string): string {
  const $ = cheerio.load(html);
  // Remove noise
  $('script, style, nav, footer, .ads, .cookie-banner').remove();
  // Collapse whitespace
  return $.html().replace(/\s{2,}/g, ' ').substring(0, 20000);  // Token limit
}
```

---

## PART 3 — ETL PIPELINE PATTERNS

### 3.1 Pipeline Architecture

```typescript
// Generic ETL pipeline with error handling + progress tracking
interface PipelineResult<T> {
  success: T[];
  failed: Array<{ item: unknown; error: string }>;
  stats: { total: number; succeeded: number; failed: number; durationMs: number };
}

async function runPipeline<TRaw, TTransformed>(
  extract: () => Promise<TRaw[]>,
  transform: (item: TRaw) => Promise<TTransformed | null>,
  load: (items: TTransformed[]) => Promise<void>,
  options: { batchSize?: number; concurrency?: number } = {}
): Promise<PipelineResult<TTransformed>> {
  const { batchSize = 100, concurrency = 5 } = options;
  const start = Date.now();

  // EXTRACT
  console.log('Extracting data...');
  const rawItems = await extract();
  console.log(`Extracted ${rawItems.length} items`);

  // TRANSFORM (with concurrency control)
  console.log('Transforming...');
  const success: TTransformed[] = [];
  const failed: Array<{ item: unknown; error: string }> = [];

  // Process in chunks to control concurrency
  for (let i = 0; i < rawItems.length; i += concurrency) {
    const chunk = rawItems.slice(i, i + concurrency);
    const results = await Promise.allSettled(chunk.map(item => transform(item)));

    for (let j = 0; j < results.length; j++) {
      const result = results[j];
      if (result.status === 'fulfilled' && result.value !== null) {
        success.push(result.value);
      } else {
        failed.push({
          item: chunk[j],
          error: result.status === 'rejected' ? result.reason?.message : 'Transform returned null',
        });
      }
    }
  }

  // LOAD in batches
  console.log(`Loading ${success.length} items...`);
  for (let i = 0; i < success.length; i += batchSize) {
    await load(success.slice(i, i + batchSize));
  }

  const stats = {
    total: rawItems.length,
    succeeded: success.length,
    failed: failed.length,
    durationMs: Date.now() - start,
  };

  console.log(`Pipeline complete: ${JSON.stringify(stats)}`);
  return { success, failed, stats };
}
```

### 3.2 Change Detection (Price/Content Monitor)

```typescript
// Monitor a URL for changes and alert when content differs
export async function monitorForChanges(
  url: string,
  selector: string,   // CSS selector for monitored element
  env: Env
): Promise<{ changed: boolean; oldValue: string | null; newValue: string }> {
  // Fetch current content
  const response = await fetch(url, {
    headers: { 'User-Agent': 'Mozilla/5.0 (compatible; MyMonitor/1.0)' }
  });
  const html = await response.text();
  const $ = cheerio.load(html);
  const currentValue = $(selector).text().trim();

  // Compare with stored value
  const cacheKey = `monitor:${encodeURIComponent(url)}:${selector}`;
  const previousValue = await env.KV.get(cacheKey);

  const changed = previousValue !== null && previousValue !== currentValue;

  // Update stored value
  await env.KV.put(cacheKey, currentValue, { expirationTtl: 86400 * 30 });  // 30 days

  return { changed, oldValue: previousValue, newValue: currentValue };
}
```

### 3.3 Cron-Based Data Collection Worker

```typescript
// wrangler.toml: crons = ["0 */6 * * *"]  (every 6 hours)
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext): Promise<void> {
    ctx.waitUntil(runDataCollection(env));
  }
};

async function runDataCollection(env: Env): Promise<void> {
  const job = await createJobRecord(env);

  try {
    const urls = await getUrlsToScrape(env);

    // Rate-limited batch processing
    const rateLimiter = new RateLimiter({ requestsPerSecond: 2 });
    const results: ScrapedData[] = [];

    for (const url of urls) {
      await rateLimiter.throttle();
      try {
        const data = await scrapeUrl(url, env);
        if (data) results.push(data);
      } catch (error) {
        console.error(`Failed to scrape ${url}:`, error);
        await logScrapeError(url, error, env);
      }
    }

    // Batch insert into D1
    if (results.length > 0) {
      const statements = results.map(r =>
        env.DB.prepare('INSERT OR REPLACE INTO scraped_data VALUES (?, ?, ?, ?)')
          .bind(r.id, r.url, JSON.stringify(r.data), r.scrapedAt)
      );
      await env.DB.batch(statements);
    }

    await completeJobRecord(job.id, results.length, env);
    console.log(`Collected ${results.length} records`);
  } catch (error) {
    await failJobRecord(job.id, error, env);
    throw error;
  }
}

// Simple rate limiter
class RateLimiter {
  private tokens: number;
  private lastRefill: number;
  private readonly rate: number;

  constructor({ requestsPerSecond }: { requestsPerSecond: number }) {
    this.rate = requestsPerSecond;
    this.tokens = requestsPerSecond;
    this.lastRefill = Date.now();
  }

  async throttle(): Promise<void> {
    this.refill();
    if (this.tokens < 1) {
      const waitMs = (1 / this.rate) * 1000;
      await new Promise(resolve => setTimeout(resolve, waitMs));
      this.refill();
    }
    this.tokens--;
  }

  private refill(): void {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.rate, this.tokens + elapsed * this.rate);
    this.lastRefill = now;
  }
}
```

---

## PART 4 — DATA CLEANING & NORMALIZATION

```typescript
// Universal data normalization utilities
export const normalize = {
  // Remove non-printable characters and normalize whitespace
  text: (s: string): string =>
    s.replace(/[\x00-\x1F\x7F-\x9F]/g, '').replace(/\s+/g, ' ').trim(),

  // Parse various date formats to ISO string
  date: (dateStr: string): string | null => {
    try {
      const date = new Date(dateStr);
      if (isNaN(date.getTime())) return null;
      return date.toISOString();
    } catch {
      return null;
    }
  },

  // Parse price from various formats
  priceCents: (priceStr: string): number | null => {
    const cleaned = priceStr.replace(/[^0-9.,]/g, '').replace(',', '.');
    const value = parseFloat(cleaned);
    if (isNaN(value)) return null;
    return Math.round(value * 100);
  },

  // Normalize URLs
  url: (url: string, baseUrl: string): string | null => {
    try {
      return new URL(url, baseUrl).href;
    } catch {
      return null;
    }
  },

  // Extract domain from URL
  domain: (url: string): string | null => {
    try {
      return new URL(url).hostname.replace('www.', '');
    } catch {
      return null;
    }
  },

  // Remove HTML tags
  stripHtml: (html: string): string =>
    html.replace(/<[^>]*>/g, ' ').replace(/&[a-z]+;/gi, ' ').replace(/\s+/g, ' ').trim(),
};

// Validate and coerce a scraped record against a schema
export function validateRecord<T>(
  record: unknown,
  schema: Record<string, 'string' | 'number' | 'boolean' | 'date' | 'url'>
): T | null {
  if (!record || typeof record !== 'object') return null;

  const result: Record<string, unknown> = {};

  for (const [key, type] of Object.entries(schema)) {
    const value = (record as Record<string, unknown>)[key];
    if (value === undefined || value === null) continue;

    switch (type) {
      case 'string':  result[key] = normalize.text(String(value)); break;
      case 'number':  result[key] = Number(value); if (isNaN(result[key] as number)) return null; break;
      case 'boolean': result[key] = Boolean(value); break;
      case 'date':    result[key] = normalize.date(String(value)); break;
      case 'url':     result[key] = normalize.url(String(value), ''); break;
    }
  }

  return result as T;
}
```

---

## PART 5 — RESPONSIBLE SCRAPING GUIDELINES

```
Legal & Ethical:
□ Check robots.txt before scraping: fetch /robots.txt and parse Disallow rules
□ Review Terms of Service for scraping prohibitions
□ Never scrape personal data without consent (GDPR / privacy laws)
□ Don't scrape and redistribute copyrighted content commercially
□ Identify your bot with a custom User-Agent and contact email

Technical Etiquette:
□ Rate limit: max 1-2 requests/second per domain (unless ToS allows more)
□ Respect Retry-After header on 429 responses
□ Use conditional GET (If-Modified-Since / ETag) to avoid re-downloading unchanged content
□ Scrape during off-peak hours (nights, weekends) for heavy jobs
□ Cache aggressively — never re-fetch data you already have

Resilience:
□ Handle HTTP errors gracefully (404, 429, 503) — don't crash pipeline
□ Detect layout changes (count of extracted items drops → alert + pause)
□ Store raw HTML snapshots for debugging (R2, 7-day TTL)
□ Test selectors against multiple pages (not just one sample)
□ Separate extraction (brittle) from transformation (stable)
```

---

> **Core Principle**: Scraping is a means to an end — the data matters, not the scraper.
> Build extractors that fail loudly, transform data idempotently, and always leave
> the source site in the same state you found it.
