# Implementation Plan & Milestones: SafeOrbit

## 1. Phased Roadmap

The development of SafeOrbit will follow a phased approach, allowing us to deliver value incrementally and gather feedback at each stage.

### Phase 1: MVP (Discovery + Read-Only Scan + Advisory)

*   **Features:**
    *   User registration and domain onboarding.
    *   Comprehensive, read-only scanning of DNS, HTTPS, and basic security.
    *   A detailed health snapshot with clear explanations and recommendations.
*   **Dependencies:**
    *   Core Supabase schema for users, domains, and scan results.
    *   Vercel frontend for user interface.
    *   Integrations with DNS and WHOIS lookup services.
*   **Success Criteria:**
    *   Users can successfully onboard a domain and view a detailed, accurate health report.

### Phase 2: Safe Auto-Fixes for DNS & HTTPS + Site Optimization

*   **Features:**
    *   Guided setup for delegating NS records.
    *   Automated HTTPS provisioning and renewal.
    *   One-click fixes for common, low-risk DNS issues (e.g., adding an A record).
    *   **AI-Powered Site Optimization:**
        *   Automatic technology stack detection (CMS, hosting, framework, language).
        *   Site analysis across 6 categories (Performance, Security, SEO, Accessibility, Best Practices, UX).
        *   Priority-ranked recommendations with step-by-step instructions.
        *   Technology-specific optimization guides (WordPress, Shopify, Next.js, etc.).
        *   One-click auto-fix for safe optimizations (caching, security headers, lazy loading).
        *   0-100 scoring system with visual dashboard.
*   **Dependencies:**
    *   Integration with a DNS provider (e.g., Cloudflare) for automated record management.
    *   Integration with Let's Encrypt for SSL certificates.
    *   Technology detection APIs (Wappalyzer, BuiltWith, or custom detection).
    *   AI/LLM integration for generating tailored recommendations.
    *   Performance testing tools (Lighthouse, PageSpeed Insights API).
*   **Success Criteria:**
    *   Users can successfully enable HTTPS and fix common DNS issues with a single click.
    *   Users can connect their site and receive a detailed optimization report within 60 seconds.
    *   At least 80% accuracy in technology stack detection.
    *   Users can apply at least 3 safe auto-fixes without manual intervention.

### Phase 3: Provider WAF Integration (Cloudflare, Bunny)

*   **Features:**
    *   Integration with Cloudflare and/or Bunny for WAF and security rule management.
    *   Pre-configured security profiles (Relaxed, Balanced, Strict).
    *   A simple interface for users to select a profile and enable/disable security features.
*   **Dependencies:**
    *   API integrations with Cloudflare and Bunny.
*   **Success Criteria:**
    *   Users can successfully apply a security profile to their domain and see the effects (e.g., blocked threats).

### Phase 4: CMS Health Checks and Patching via SSH

*   **Features:**
    *   Optional SSH-based connection to a user's server.
    *   Automated detection of CMS (WordPress first) and installed plugins.
    *   Vulnerability scanning and one-click updating with backup and rollback.
*   **Dependencies:**
    *   A secure way to store and manage SSH credentials.
    *   A reliable mechanism for creating backups and performing updates.
*   **Success Criteria:**
    *   Users can successfully connect their WordPress site and perform a one-click update of a vulnerable plugin.
