# 🛡️ MirApi Proxy Gateway

[![Website](https://img.shields.io/badge/Website-mirapi.io-blue?style=flat-square)](https://mirapi.io)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)](https://github.com/MirApiGateway/mirapi/issues)

> **The High-Reliability API Proxy Gateway for Lazy but Smart Developers.**  
> Supercharge your third-party API integrations (Stripe, payment gateways, banking APIs, AI providers) with zero code changes. Simply point your requests to the proxy and use custom HTTP headers to instantly add enterprise-grade reliability.

---

## 🎁 Free Forever Tier (No Card Required)
To help you build, test, and run production workloads risk-free, MirApi offers a **Free Forever Plan**:
* **10,000 free requests per month** (resets monthly).
* **No credit card required** to sign up.
* Hosted on our high-performance **Singapore Node** yielding **< 2ms** internal processing latency.
* Register in 10 seconds at [mirapi.io](https://mirapi.io) and start testing immediately using our public **Mock Server** at `https://mock.mirapi.io`.

---

## 🎯 Production Use Cases & Solutions

### 1. India: UPI Failover & Cascade Routing (`X-Route-Key`)
* **The Problem:** Unified Payments Interface (UPI) gateways in India (like Razorpay, Paytm) frequently face bank-side timeouts, causing dropped checkouts and lost revenue.
* **The Solution:** Define a database-backed cascade route. If the primary gateway fails to respond within a tight execution limit (e.g. `X-Attempt-Timeout: 2000ms`), MirApi silently and automatically cascades (`priority` or concurrent `race` strategies) to a backup gateway like Cashfree or Paytm. Zero changes to your backend application logic.

### 2. Shopify: Webhook Delivery Queue & Field Mapping (`X-Webhook-Callback`)
* **The Problem:** Shopify webhooks are dropped if your CRM/ERP system is down. In addition, Shopify sends a standard payload, while your destination API might require a different structure, forcing you to write and maintain custom middleware.
* **The Solution:**
  * Route Shopify webhooks through MirApi using `X-Webhook-Callback: https://your-erp.com/orders`.
  * Transform raw Shopify JSON schemas at the edge using `X-Extract-Map` rules (e.g. mapping `$.id=>order_id`) to fit your destination API structure.
  * If your ERP goes down, MirApi queues the webhook in a Redis-backed queue and retries delivery automatically for up to 24 hours.

---

## 🛠️ Quick Start & Integration Examples

Integrating MirApi requires **no SDKs**. Simply redirect your API endpoint through our gateway (`https://proxy.mirapi.io/`) and append custom orchestration headers.

### 1. Basic Routing with Auto-Retries & Jitter
Forward requests to your target API and automatically retry up to 5 times using **Exponential Backoff and Jitter** in case of upstream errors (HTTP `5xx` or timeouts).

```bash
curl -X POST https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: YOUR_MIRAPI_KEY" \
  -H "X-Target-URL: https://api.stripe.com/v1/charges" \
  -H "X-Retry-Count: 5" \
  -H "X-Retry-Delay: 300ms" \
  -H "Content-Type: application/json" \
  -d '{"amount": 1000, "currency": "usd"}'
```

### 2. Smart Cache (Fallback Caching on Outage)
If the upstream target system collapses or returns a catastrophic `5xx` server error, MirApi will intercept the error and serve the newest stable cached response from memory.

```bash
curl -X GET https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: YOUR_MIRAPI_KEY" \
  -H "X-Target-URL: https://api.weatherapi.com/v1/current.json?q=Singapore" \
  -H "X-Smart-Cache: 300s"
```

### 3. Asynchronous Webhooks Execution
Release client threads instantly with a `202 Accepted` packet. MirApi handles background execution via worker queues and pushes the resulting body to your webhook receiver.

```bash
curl -X POST https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: YOUR_MIRAPI_KEY" \
  -H "X-Target-URL: https://api.openai.com/v1/chat/completions" \
  -H "X-Webhook-Callback: https://your-app.com/webhooks/openai" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-4", "messages": [{"role": "user", "content": "Hello!"}]}'
```

### 4. PCI-DSS Compliance (Hard Card Data Blocker)
Prevent accidental raw card data leaks. MirApi validates all incoming request bodies using the **Luhn Algorithm**. If raw card numbers are detected, the request is blocked immediately at the edge.

```bash
# This request gets blocked:
curl -X POST https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: YOUR_MIRAPI_KEY" \
  -H "X-Target-URL: https://api.stripe.com/v1/charges" \
  -H "Content-Type: application/json" \
  -d '{"card_number": "4111111111111111"}'
```

* **Response Status:** `400 Bad Request`
* **Response Body:** `"Error: Clear-text payment data detected. Please use tokenization (Stripe/Braintree tokens) instead of raw card numbers."`

---

## 🧪 Testing with the Public Mock Server

We host a public mock server at `https://mock.mirapi.io` specifically for testing and playground verification. You can route proxy requests through MirApi to simulate various success and failure targets instantly.

### Mock Endpoints Available:

| Upstream Path | Behavior & Test Scenario |
| :--- | :--- |
| `GET/POST https://mock.mirapi.io/post` | Mirrors input JSON payload back to you (Standard POST, Payload Mapping). |
| `GET https://mock.mirapi.io/delay/:seconds` | Intentionally delays response to test timeouts (e.g. `X-Attempt-Timeout`). |
| `GET https://mock.mirapi.io/status/:code` | Returns the specified HTTP status code (e.g., `502`, `503` to test Retries & Failovers). |
| `GET https://mock.mirapi.io/random` | Returns a randomized `200 OK` or `502 Bad Gateway` (perfect for testing retry loops). |

### Test Example: Auto-Retry Loop with Mock Server
Test the retry behavior safely. The proxy will catch the mock server's `502` status errors and automatically retry until it gets a successful response:

```bash
curl -X POST https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: YOUR_MIRAPI_KEY" \
  -H "X-Target-URL: https://mock.mirapi.io/random" \
  -H "X-Retry-Count: 4" \
  -H "X-Retry-Delay: 200ms"
```

---

## 🔒 Security & Secrets Offloading

Keep API keys out of configuration files and frontend code:
* **Ephemeral Storage (`X-Identity-Key`):** API keys are read directly into memory loops, leveraged during processing runtime, and scrubbed clean without logging.
* **Encrypted Storage (`X-Proxy-Master-Key`):** Decrypt database-secured API credentials dynamically at the edge using the cryptographic passphrase sent in this header.

---

## 💬 Feedback, Support & Bug Reports

We want to make third-party integrations as reliable as possible! 

* **Found a bug?** Let us know by opening an issue.
* **Have a feature request?** Feel free to submit an issue outlining your proposal.
* **Questions or Suggestions?** Join the discussion in the issues tracker.

Go to the [Issues](https://github.com/MirApiGateway/mirapi/issues) section to submit your feedback.

---

*For full API documentation, dashboard usage, and deployment guides, visit the official website at [mirapi.io](https://mirapi.io).*
