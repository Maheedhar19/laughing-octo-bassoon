# Claude + Fabric SQL Warehouse — Assessment and Recommendation

I've explored the approach for connecting Claude to our Fabric SQL Warehouse, so business users could query it directly through natural language. Sharing what I found and my recommendation below.

## Core Issues Identified

### 1. No native connector

Claude doesn't currently offer a built-in Fabric connector, so we'd need to either build and maintain our own hosted integration, or use a third-party managed service. Either way, this is an ongoing engineering and cost commitment, not a one-time setup.

### 2. Data sensitivity rules out the managed-vendor route

Since this is confidential pharma data, routing it through a third-party hosted MCP service means that vendor sits in the data path between our warehouse and Claude — our internal architecture/security team is unlikely to approve this.

That leaves only the self-hosted option, which is the more resource-intensive path of the two, with us owning all the infrastructure, security, and maintenance.

### 3. Open-ended access means heavier security work

Since business users need to ask anything rather than work from a fixed set of reports, we can't scope this down to a handful of safe, pre-validated views.

That pushes real security work onto us — read-only access controls, row- and column-level security, audit logging — just to make open-ended querying safe across multiple users, on top of the confidentiality requirements above.

### 4. Accuracy still isn't guaranteed

Even with all of the above in place, natural-language-to-SQL has known failure modes: the model can miscount, apply implicit limits, or misread ambiguous terms in the data.

For business users making decisions off these answers, that's a real risk we can't fully engineer around.

## High-Level Resources Required (Azure)

Assuming a self-hosted approach given the data sensitivity:

### Identity

- Azure AD (Entra ID) app registration, for OAuth between the connector and Fabric
- A dedicated read-only SQL login or service principal (SELECT-only, no write/DDL access)
- Entra ID groups to manage and scope which users get access

### Hosting

- Azure App Service, Container Apps, or Functions, to host the server as a public HTTPS endpoint
- Azure Key Vault, to store credentials securely
- Azure API Management or Application Gateway, for rate limiting and centralized logging
- Azure Monitor / Log Analytics, for audit logging of who queried what
- A custom domain and TLS certificate for a stable public endpoint
- Likely a security/architecture review given the confidential nature of the data, before this could go live
