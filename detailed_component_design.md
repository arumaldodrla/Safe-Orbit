# Detailed Component Design: SafeOrbit

## 1. Introduction

This document provides a detailed design for each major component of the SafeOrbit platform. Each component is designed to be modular, scalable, and maintainable, with clear responsibilities and boundaries.

## 2. Supabase Database & Auth

*   **Responsibilities:**
    *   User and organization management (multi-tenancy).
    *   Authentication and authorization.
    *   Persistence of all platform data, including domains, DNS snapshots, scan results, and provider configurations.
*   **API Surface:**
    *   The database will be accessed primarily through the Supabase client library and custom Postgres functions.
    *   Row-Level Security (RLS) will be used extensively to enforce data access policies.
*   **Data Models:** See the `data_model_schema.md` document for a detailed schema.
*   **Failure Modes:**
    *   **Database Unavailability:** The platform will be unavailable. Monitoring and alerting will be in place to detect and respond to database outages.
    *   **Data Corruption:** Regular backups will be taken to allow for point-in-time recovery.

## 3. Scan Orchestrator (Supabase Functions)

*   **Responsibilities:**
    *   Orchestrating the process of scanning a domain.
    *   Calling various external and internal services to perform specific scan tasks (e.g., DNS lookup, HTTPS check).
    *   Aggregating the results and storing them in the database.
*   **API Surface:**
    *   The orchestrator will be implemented as a set of Supabase Edge Functions.
    *   It will be triggered by API calls from the frontend or by a scheduler.
*   **Failure Modes:**
    *   **Function Timeout:** Scans may time out for large or slow domains. The orchestrator will be designed to be resumable.
    *   **External Service Failure:** The orchestrator will handle failures of external services gracefully, logging the error and retrying if appropriate.

## 4. DNS Service Integrations

*   **Responsibilities:**
    *   Providing a unified interface for interacting with different DNS providers (Cloudflare, Bunny, etc.).
    *   Translating SafeOrbit's internal DNS record format to the provider-specific format.
*   **API Surface:**
    *   A generic `DnsProvider` interface will be defined, with concrete implementations for each supported provider.
*   **Failure Modes:**
    *   **Provider API Unavailability:** The platform will not be able to manage DNS records for that provider. The UI will reflect the provider's status.
    *   **Invalid API Credentials:** The platform will detect invalid credentials and prompt the user to update them.

## 5. Security Policy Engine

*   **Responsibilities:**
    *   Encoding the rules and recommendations for what constitutes a "secure" configuration.
    *   Analyzing scan results and generating findings and recommendations.
    *   Determining whether a fix can be applied automatically or requires user approval.
*   **API Surface:**
    *   The engine will be implemented as a library or a set of functions that can be called by the scan orchestrator and the AI orchestration layer.
*   **Failure Modes:**
    *   **Incorrect Recommendation:** The policy engine is a critical component, and its logic will be thoroughly tested to avoid making incorrect recommendations.

## 6. AI Orchestration Layer

*   **Responsibilities:**
    *   Providing the "brain" of the platform.
    *   Making decisions about when to scan, what to fix, and how to interact with the user.
    *   Interacting with the other components of the system via APIs and the MCP protocol.
*   **API Surface:**
    *   The AI layer will expose a set of tools that can be called by AI models (e.g., via function-calling).
    *   These tools will provide a safe and controlled way for the AI to interact with the platform.
*   **Failure Modes:**
    *   **Erroneous AI Decision:** The AI's actions will be constrained by the tools it has access to, and all critical operations will require human approval.

## 7. Job Scheduler

*   **Responsibilities:**
    *   Scheduling and running periodic tasks, such as security scans and certificate renewals.
*   **API Surface:**
    *   The scheduler will be implemented using a cron-based system (e.g., Supabase's pg_cron or a Vercel cron job).
*   **Failure Modes:**
    *   **Missed Job:** The scheduler will be monitored to ensure that jobs are being executed as expected.
