# System Architecture Overview: SafeOrbit

## 1. High-Level Architecture

The SafeOrbit platform is designed as a modern, scalable, and maintainable system that leverages a serverless architecture and managed services to minimize operational overhead. The architecture is composed of four main layers:

1.  **Frontend (Vercel):** A React-based web application built with Refine for the admin-style interface, deployed on Vercel.
2.  **Backend (Supabase):** A Postgres database with authentication, file storage, and serverless functions, all provided by Supabase.
3.  **External Providers:** A set of third-party services that provide the core functionality for DNS, security, and CDN, such as Cloudflare and Bunny.
4.  **AI/MCP Orchestration Layer:** An AI-driven layer that automates tasks, makes decisions, and interacts with the other layers of the system.

### 1.1. Architecture Diagram

```
+----------------------------------------------------------------------------------------------------+
|                                            User (Browser)                                          |
+----------------------------------------------------------------------------------------------------+
     |                                                                                             ^
     | HTTPS                                                                                         |
     v                                                                                             |
+----------------------------------------------------------------------------------------------------+
|                                       Frontend (Vercel)                                            |
|                                                                                                    |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
|    |      React Web App      |      |   Refine Admin Panel    |      |   Vercel Edge Functions |   |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
+----------------------------------------------------------------------------------------------------+
     |                                           ^                                  ^
     | API Calls (REST/GraphQL)                  |                                  | API Calls
     v                                           |                                  v
+----------------------------------------------------------------------------------------------------+
|                                       Backend (Supabase)                                           |
|                                                                                                    |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
|    |   Postgres Database     |      |   Supabase Auth         |      | Supabase Edge Functions |   |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
+----------------------------------------------------------------------------------------------------+
     |                                                                                             ^
     | API Calls                                                                                     |
     v                                                                                             |
+----------------------------------------------------------------------------------------------------+
|                                   External Providers                                               |
|                                                                                                    |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
|    |      Cloudflare         |      |         Bunny           |      |      Other DNS/WAF      |   |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
+----------------------------------------------------------------------------------------------------+
     ^                                                                                             |
     | Instructions/API Calls                                                                        |
     v                                                                                             |
+----------------------------------------------------------------------------------------------------+
|                                   AI/MCP Orchestration Layer                                       |
|                                                                                                    |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
|    |      Manus Agents       |      |      AI Models          |      |      Tooling/MCP        |   |
|    +-------------------------+      +-------------------------+      +-------------------------+   |
+----------------------------------------------------------------------------------------------------+
```

## 2. Data Flow for Key Operations

### 2.1. Initial Scan of a New Domain

1.  The user enters a domain name in the frontend.
2.  The frontend calls a Supabase Edge Function to initiate the scan.
3.  The Supabase function, orchestrated by the AI layer, makes a series of API calls to external services to gather information about the domain (DNS records, HTTPS status, etc.).
4.  The results are stored in the Supabase Postgres database.
5.  The frontend queries the database and displays the health snapshot to the user.

### 2.2. Changing DNS Records

1.  The user modifies a DNS record in the Refine admin panel.
2.  The frontend sends the change request to a Supabase Edge Function.
3.  The AI orchestration layer validates the change, checking for potential conflicts or issues.
4.  If the change is safe, the AI layer instructs the appropriate provider (e.g., Cloudflare) via its API to update the DNS record.
5.  The change is logged in the Supabase database for auditing and rollback.

### 2.3. Enabling HTTPS and Security Features

1.  The user enables a security feature, such as "Force HTTPS," from the dashboard.
2.  The frontend sends the request to the backend.
3.  The AI orchestration layer determines the necessary steps to enable the feature (e.g., creating a redirect rule in Cloudflare).
4.  The AI layer executes these steps by calling the provider APIs.
5.  The new configuration is saved in the Supabase database.

### 2.4. Running Periodic Security Scans

1.  A scheduled job (e.g., a cron job) triggers a Supabase Edge Function.
2.  The function, guided by the AI layer, performs a routine security scan of a domain.
3.  If any issues are found, they are stored in the database as "findings."
4.  The AI layer can then either automatically remediate the issue or create a recommendation for the user to review, depending on the severity and risk of the issue and the user's preferences.
