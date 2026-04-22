# Deployment Guide

This document covers safe deployment options for the Instrument Innovation Agent. The core principle: **the `ANTHROPIC_API_KEY` must never reach the client browser**.

---

## Option 1: Cloudflare Worker Proxy (recommended)

```js
// worker.js
export default {
  async fetch(request, env) {
    // SECURITY: Only accept POST from your allowed origin
    const origin = request.headers.get('Origin') || '';
    if (!env.ALLOWED_ORIGINS.split(',').includes(origin)) {
      return new Response('Forbidden', { status: 403 });
    }

    if (request.method !== 'POST') {
      return new Response('Method Not Allowed', { status: 405 });
    }

    // SECURITY: Rate limiting should be added via Cloudflare Rate Limiting rules

    const body = await request.json();

    const upstream = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': env.ANTHROPIC_API_KEY,  // injected from Worker secret
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify(body),
    });

    // Stream the response back
    return new Response(upstream.body, {
      status: upstream.status,
      headers: {
        'Content-Type': 'text/event-stream',
        'Access-Control-Allow-Origin': origin,
      },
    });
  },
};
```

Set secrets in Cloudflare dashboard (never in `wrangler.toml`):
```bash
wrangler secret put ANTHROPIC_API_KEY
wrangler secret put ALLOWED_ORIGINS
```

---

## Option 2: Express.js Proxy

```js
// proxy/server.js
'use strict';

const express    = require('express');
const rateLimit  = require('express-rate-limit');
const fetch      = require('node-fetch');
require('dotenv').config();

const app  = express();
const PORT = process.env.PORT || 3001;

// SECURITY: Parse JSON bodies with size limit to prevent payload attacks
app.use(express.json({ limit: '32kb' }));

// SECURITY: Rate limit — 10 requests per minute per IP
const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: parseInt(process.env.RATE_LIMIT_REQUESTS_PER_MINUTE || '10', 10),
  standardHeaders: true,
  legacyHeaders: false,
});
app.use('/v1/messages', limiter);

// SECURITY: CORS restricted to allowed origins
app.use((req, res, next) => {
  const allowed = (process.env.ALLOWED_ORIGINS || '').split(',').map(s => s.trim());
  const origin  = req.headers.origin || '';
  if (allowed.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
  }
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
  if (req.method === 'OPTIONS') return res.sendStatus(204);
  next();
});

// SECURITY: Security headers on all responses
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  next();
});

app.post('/v1/messages', async (req, res) => {
  // SECURITY: Never log request bodies — they contain user queries
  const upstream = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      // SECURITY: Key loaded from environment variable, never from source code
      'x-api-key': process.env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01',
    },
    body: JSON.stringify(req.body),
  });

  res.status(upstream.status);
  res.setHeader('Content-Type', 'text/event-stream');
  upstream.body.pipe(res);
});

app.listen(PORT, () => {
  console.log(`Proxy listening on port ${PORT}`);
});
```

Install dependencies:
```bash
npm install express express-rate-limit node-fetch dotenv
```

---

## Static Hosting (Netlify / Vercel)

After deploying a proxy, update `API_ENDPOINT` in `src/index.html`:

```js
const API_ENDPOINT = 'https://your-proxy.netlify.app/v1/messages';
```

Serve `src/index.html` from any static host. No build step required.

---

## Local Development

```bash
# Serve the static file on port 8080
python3 -m http.server 8080 --directory src/

# Or with Node.js
npx serve src/ -p 8080
```

For local dev, `API_ENDPOINT` can point directly to `https://api.anthropic.com/v1/messages`.  
Set your key in `.env` — the static file does not auto-load `.env`; pass the key via your proxy or a build step.

> ⚠️ Direct browser → Anthropic calls expose your key in DevTools network tab. Use only on a trusted local machine.
