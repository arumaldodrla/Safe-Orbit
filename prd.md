# Product Requirements Document (PRD): SafeOrbit

## 1. Problem Statement

Website owners, from small businesses to individual creators, face a significant and persistent challenge in managing the technical complexities of DNS, HTTPS, and web security. Misconfigurations are common, leading to downtime, security vulnerabilities, and poor user experience. The landscape of tools is fragmented and often requires specialized knowledge, which is a major barrier for non-technical users. SafeOrbit aims to solve these problems by providing a unified, AI-driven platform that acts as a "DNS + Web Security Copilot," making it easy for anyone to achieve a secure and reliable online presence.

## 2. Target Users & Personas

We will build SafeOrbit to serve three primary user personas:

| Persona | Description | Key Needs |
| :--- | :--- | :--- |
| **Savvy Admin** | Technically proficient, comfortable with DNS and security concepts. | Advanced controls, detailed logs, automation, and efficiency. |
| **Non-Technical but Comfortable User** | Small business owner or marketer who can follow guided instructions. | Simplicity, clear explanations, and low-risk, guided actions. |
| **"I Don’t Know How I Got Here" User** | Low technical literacy, needs a simple, clear status. | At-a-glance status (e.g., red/yellow/green) and one-click safe fixes. |

## 3. Goals & Success Metrics

| Goal | Success Metric |
| :--- | :--- |
| **Simplify DNS and Security Management** | - Reduction in support tickets related to DNS/security issues.<br>- High user satisfaction scores (CSAT/NPS). |
| **Improve Website Security Posture** | - Increase in the number of sites with correctly configured SPF/DKIM/DMARC.<br>- Reduction in common security misconfigurations (e.g., missing security headers). |
| **Automate Repetitive Tasks** | - High adoption rate of automated features (e.g., auto-patching, auto-remediation).<br>- Reduction in the time users spend on manual configuration. |
| **Achieve High Platform Adoption** | - Growth in the number of active users and domains managed by the platform. |

## 4. Core Features

### 4.1. Smart Onboarding & Discovery

*   **User Story (As a new user):** I want to enter my domain name and get a clear, easy-to-understand health check of my website's DNS, HTTPS, and security, so I can quickly identify any issues.
*   **Acceptance Criteria:**
    *   The platform must accept a domain name as input.
    *   It must scan for and display current DNS records (A/AAAA, MX, CNAME, TXT, SPF, DKIM, DMARC, CAA, NS).
    *   It must check for HTTPS and common security headers.
    *   It should attempt to identify the CMS (WordPress, Joomla, etc.).
    *   The results must be presented in a human-readable "health snapshot."
*   **Out of Scope (v1):** Deep malware scanning, performance analysis.

### 4.2. AI DNS Copilot

*   **User Story (As a Savvy Admin):** I want to manage all my DNS records in one place, with templates for common services and guardrails to prevent me from making critical mistakes.
*   **Acceptance Criteria:**
    *   The platform must allow users to manage DNS records through either NS delegation or API integration with their existing provider.
    *   It must provide pre-configured templates for common setups (e.g., Google Workspace, Microsoft 365).
    *   It must warn users before they make a change that could break their website or email.
    *   It must visualize DNS propagation status.
*   **Out of Scope (v1):** Support for all DNS providers; only a few major ones will be supported initially.

### 4.3. HTTPS / TLS & Mixed-Content Manager

*   **User Story (As a non-technical user):** I want the platform to automatically set up HTTPS for my site and fix any mixed-content issues, so my visitors see a secure lock icon in their browser.
*   **Acceptance Criteria:**
    *   The platform must automate the provisioning and renewal of SSL/TLS certificates (e.g., via Let's Encrypt).
    *   It must automatically redirect all HTTP traffic to HTTPS.
    *   It must detect and provide a way to fix mixed-content warnings.
*   **Out of Scope (v1):** Management of custom or EV certificates.

### 4.4. Edge Security Layer

*   **User Story (As a business owner):** I want to protect my site from common attacks like bots and brute-force attempts, without needing to be a security expert.
*   **Acceptance Criteria:**
    *   The platform must allow users to configure a Web Application Firewall (WAF) and rate-limiting through a simple interface.
    *   It must provide pre-configured security profiles (e.g., Relaxed, Balanced, Strict).
    *   It must be able to configure security headers.
*   **Out of Scope (v1):** Advanced DDoS mitigation for large-scale attacks.

### 4.5. CMS Health & Auto-Patching

*   **User Story (As a user with a WordPress site):** I want the platform to tell me when my plugins are out of date and give me a one-click option to update them safely.
*   **Acceptance Criteria:**
    *   The platform must be able to connect to a user's site via SSH or an agent.
    *   It must identify the CMS, plugins, and themes.
    *   It must check for known vulnerabilities and available updates.
    *   It must offer a one-click update feature with automated backups and rollback.
*   **Out of Scope (v1):** Support for all CMSs; initial focus on WordPress.

### 4.6. Email Authentication & Deliverability Panel

*   **User Story (As a marketer):** I want to make sure my emails are being delivered and that no one can spoof my domain, and I want to understand the reports.
*   **Acceptance Criteria:**
    *   The platform must provide a guided setup for SPF, DKIM, and DMARC.
    *   It must monitor and parse DMARC reports, presenting them in an easy-to-understand format.
    *   It must provide a "spoof-resistance score" and actionable recommendations.
*   **Out of Scope (v1):** Advanced email deliverability analytics (e.g., inbox placement).
