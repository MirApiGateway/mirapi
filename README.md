# 🛡️ MirApi Proxy Gateway

[![Website](https://img.shields.io/badge/Website-mirapi.io-blue?style=flat-square)](https://mirapi.io)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)](https://github.com/MirApiGateway/mirapi/issues)

> **The High-Reliability API Proxy Gateway for Lazy but Smart Developers.**  
> Supercharge your third-party API integrations (Stripe, payment gateways, banking APIs, AI providers) with zero code changes. Simply point your requests to the proxy and use custom HTTP headers to instantly add enterprise-grade reliability.

---

## 🎯 What is MirApi?

**MirApi** is an ultra-low latency, high-performance API proxy gateway written in Go. It sits between your application and upstream services (like Stripe, OpenAI, or banking APIs). By changing only the base URL of your API requests and injecting custom headers, you gain immediate access to advanced resilience, security, routing, and caching patterns.

### ⚡ Key Latency Overhead
* Internal processing latency: **< 2ms** per request.
* Built with a non-blocking asynchronous architecture using Go's native standard library and high-speed Redis caching.

---

## 🚀 Key Features

### 1. 🛡️ PCI-DSS Pre-Validation (Hard Stop)
Protects your infrastructure from heavy compliance liabilities. The gateway scans raw incoming JSON request bodies for 13-16 digit card numbers. If detected, it applies the **Luhn Algorithm** check and immediately blocks the request.
* *Result:* Clear-text card data never touches your persistent logs or backend servers.

### 2. 🔁 Automatic Retries & Jitter
Prevents transient network failures from breaking your application. Automatically retries failing upstream requests (`>= 500` HTTP statuses or network dropouts) using **Exponential Backoff** and randomized **Jitter** to avoid hammering target servers.

### 3. 🧠 Smart Cache (Zero-Downtime Failover)
If an upstream service goes offline or returns a `5xx` error, MirApi intercepts the failure and instantly returns the latest cached successful response from Redis.

### 4. 🔀 Database-Backed Cascade Routing & Failover
Define primary and fallback endpoints.
* **Priority Strategy:** Sequentially tries endpoints. If the primary target fails or times out, the proxy silently routes the request to a secondary provider.
* **Race Strategy:** Concurrently fires requests to multiple endpoints and resolves using the fastest successful responder, canceling all other pending requests.

### 5. ⚡ Asynchronous Webhooks (Queue Mode)
Instantly releases your client application threads by returning a `202 Accepted` status. MirApi queues the execution in a high-capacity Redis worker pool, safely executes the request (retrying for hours if needed), and posts the result back to your webhook URL.

### 6. 🔒 Secret Offloading
Keep API credentials off your application servers. MirApi can decrypt target credentials on the fly using a master passphrase passed in the headers, or read them from secure ephemeral memory loops.

---

## 💼 Where Can I Apply MirApi?

* **Payment Flow Resilience:** Prevent lost sales when Stripe, Braintree, or PayPal experience temporary outages.
* **LLM API Redundancy:** Automatically failover to a backup LLM provider (e.g., Anthropic to OpenAI) if rate limits or outages occur.
* **Legacy API Modernization:** Inject modern API features (idempotency, custom timeout limits, payload mapping, caching) into older services without rewriting their code.
* **Compliance Safeguarding:** Ensure developers or staging environments do not accidentally send clear-text credit cards to third-party endpoints.

---

## 🛠️ Minimal Examples

Integrating MirApi requires **no SDKs**. Just change the base URL to your MirApi gateway and inject the required headers.

### 1. Basic Proxying with Automatic Retries
Forward a request to httpbin (acting as your target API) and retry up to 3 times with exponential delays if it fails.

```bash
curl -X POST https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: your_mirapi_client_key" \
  -H "X-Target-URL: https://httpbin.org/post" \
  -H "X-Retry-Count: 3" \
  -H "X-Retry-Delay: 200ms" \
  -H "Content-Type: application/json" \
  -d '{"status": "resilient"}'
```

### 2. Smart Cache Rescue
Enable 5 minutes of caching. If the upstream server suddenly goes down, MirApi will serve the cached successful response.

```bash
curl -X POST https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: your_mirapi_client_key" \
  -H "X-Target-URL: https://api.stripe.com/v1/charges" \
  -H "X-Smart-Cache: 5m" \
  -H "Content-Type: application/json" \
  -d '{"amount": 2000, "currency": "usd"}'
```

### 3. Asynchronous Execution via Webhooks
Offload a long-running upstream call. MirApi immediately returns `202 Accepted` and processes the request in the background.

```bash
curl -X POST https://proxy.mirapi.io/ \
  -H "X-MirApi-Key: your_mirapi_client_key" \
  -H "X-Target-URL: https://httpbin.org/delay/10" \
  -H "X-Webhook-Callback: https://your-server.com/webhooks/mirapi" \
  -H "Content-Type: application/json" \
  -d '{"task": "background-process"}'
```

---

## 💬 Feedback, Support & Bug Reports

We want to make third-party integrations as reliable as possible! 

* **Found a bug?** Let us know by opening an issue.
* **Have a feature request?** Feel free to submit an issue outlining your proposal.
* **Questions or Suggestions?** Join the discussion in the issues tracker.

Go to the [Issues](https://github.com/your-username/mirapi/issues) section to submit your feedback.

---

*For full API documentation, dashboard usage, and deployment guides, visit the official website at [mirapi.io](https://mirapi.io).*
