# SafeOrbit: Global DNS & Web Security Copilot

## Overview

SafeOrbit is an AI-driven SaaS platform designed to simplify DNS management, HTTPS configuration, and web security for website owners of all technical levels. Acting as a "DNS + Web Security Copilot," SafeOrbit automates complex tasks, provides clear recommendations, and ensures your website is secure, reliable, and easy to manage.

## Key Features

- **Smart Onboarding & Discovery**: Automatically scan your domain to identify DNS, HTTPS, and security issues.
- **AI DNS Copilot**: Centralized DNS management with templates, guardrails, and global propagation tracking.
- **HTTPS/TLS Manager**: Automated certificate provisioning, renewal, and mixed-content detection.
- **Edge Security Layer**: Configure WAF, rate-limiting, and security headers through simple profiles.
- **CMS Health & Auto-Patching**: Optional SSH-based vulnerability scanning and one-click updates for WordPress and other CMSs.
- **Email Authentication Panel**: Guided setup for SPF, DKIM, and DMARC with continuous monitoring.
- **🚀 AI-Powered Site Optimization**: Automatically detect your website's technology stack (CMS, hosting, framework) and receive personalized, actionable recommendations across 6 key areas (Performance, Security, SEO, Accessibility, Best Practices, UX) with one-click auto-fix for safe optimizations. [Learn More](SITE_OPTIMIZATION_FEATURE.md) | [User Guide](SITE_OPTIMIZATION_USER_GUIDE.md)

## Target Users

SafeOrbit is designed for three primary personas:

1. **Savvy Admin**: Technically proficient users who want advanced controls and automation.
2. **Non-Technical but Comfortable User**: Small business owners who can follow guided steps.
3. **"I Don't Know How I Got Here" User**: Users with low technical literacy who need simple, at-a-glance status.

## Architecture

SafeOrbit is built on a modern, serverless architecture:

- **Frontend**: React + Refine, deployed on Vercel
- **Backend**: Supabase (Postgres, Auth, Edge Functions)
- **External Providers**: Cloudflare, Bunny, and other DNS/WAF providers (via API)
- **AI Orchestration**: AI agents for automated decision-making and task execution

## Documentation

This repository contains comprehensive documentation for the SafeOrbit platform:

### Core Documentation
- [Product Requirements Document (PRD)](./prd.md)
- [User Journeys & UX Flows](./user_journeys.md)
- [System Architecture Overview](./system_architecture.md)
- [Detailed Component Design](./detailed_component_design.md)
- [Data Model & Database Schema](./data_model_schema.md)
- [API Specifications](./api_specifications.md)
- [Security & Privacy Design](./security_privacy_design.md)
- [Implementation Plan & Milestones](./implementation_plan.md)
- [Deployment & DevOps Guide](./deployment_devops_guide.md)
- [Runbooks & Operational Playbooks](./runbooks_operational_playbooks.md)

### AI & Automation
- [AI-Centric Development Guidelines](./ai_centric_development_guidelines.md)
- [AI Operations & Automation](./ai_operations_automation.md)
- [AI Implementation Guide](./AI_IMPLEMENTATION_GUIDE.md)

### Features
- [Site Optimization Feature](./SITE_OPTIMIZATION_FEATURE.md)
- [Site Optimization User Guide](./SITE_OPTIMIZATION_USER_GUIDE.md)

### Market Research
- [Market Research Overview](./market-research/README.md)
- [Executive Summary](./market-research/01_EXECUTIVE_SUMMARY.md)
- [Competitive Analysis](./market-research/02_COMPETITIVE_ANALYSIS.md)
- [Customer Personas](./market-research/03_CUSTOMER_PERSONAS.md)
- [Financial Projections](./market-research/04_FINANCIAL_PROJECTIONS.md)

## Getting Started

This platform is designed to be fully developed, deployed, and maintained by AI agents. If you have no development experience, follow these steps:

1. **Review the Documentation**: Start with the [PRD](./prd.md) to understand the platform's goals and features.
2. **Use AI Tools**: Leverage AI assistants like Manus, Cursor, or Gemini to generate code based on the documentation.
3. **Deploy**: Follow the [Deployment & DevOps Guide](./deployment_devops_guide.md) to set up the platform on Vercel and Supabase.
4. **Automate**: Configure AI agents to handle routine tasks and maintenance as described in [AI Operations & Automation](./ai_operations_automation.md).

## Contributing

This project is designed to be developed and maintained primarily by AI agents. However, human oversight and approval are required for critical operations. Please refer to the [AI-Centric Development Guidelines](./ai_centric_development_guidelines.md) for more information.

## License

This project is licensed under the MIT License.

## Contact

For questions or support, please open an issue in this repository.
