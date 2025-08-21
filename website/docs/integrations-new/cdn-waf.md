---
id: cdn-waf-strategy
slug: /integrations/cdn-waf-strategy
sidebar_position: 12
---

# CDN and WAF Strategy

:::info This document outlines our strategy for using a Content Delivery Network (CDN) and Web Application Firewall (WAF) to improve performance and security. :::

:::tip Recommended Provider We recommend using **Cloudflare** for its robust CDN, WAF, and DDoS protection features. :::

## WAF Configuration

The following WAF rules should be enabled in Cloudflare:

| Rule Set | Description |
| --- | --- |
| **OWASP Core Ruleset** | Protects against common web application vulnerabilities. |
| **Cloudflare Managed Ruleset** | Provides additional protection against new and emerging threats. |

## Rate Limiting

Rate limiting should be configured to protect against brute-force attacks and other denial-of-service attacks.

| Endpoint Type | Recommendation |
| --- | --- |
| **Login endpoints** | Limit login attempts to a reasonable number per IP address. |
| **API endpoints** | Limit requests to a reasonable number per API key or IP address. |

:::note Implementation This document serves as a guideline. The specific implementation details will depend on the application and its traffic patterns. Tickets should be created to track the implementation of these recommendations. :::
