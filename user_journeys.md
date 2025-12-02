# User Journeys & UX Flows: SafeOrbit

## 1. Introduction

This document outlines the user journeys and UX flows for the three primary personas of the SafeOrbit platform. The user experience is designed to be intuitive and tailored to the technical comfort level of each persona.

## 2. Persona-Based Journeys

### 2.1. Savvy Admin

**Journey:** The Savvy Admin wants to quickly onboard a new domain, configure advanced settings, and automate as much as possible.

*   **Onboarding:**
    1.  The user signs up and is prompted to add a domain.
    2.  They enter their domain and the platform performs a quick scan.
    3.  The user is presented with the option to either delegate their NS records to SafeOrbit or connect their existing DNS provider via API.
    4.  They choose the API option and provide their Cloudflare API key.
    5.  The platform verifies the connection and imports the existing DNS records.
*   **Configuration:**
    1.  The user navigates to the DNS management panel.
    2.  They apply a pre-configured template for their email provider.
    3.  They navigate to the security panel and select the "Strict" security profile.
    4.  They review the changes and approve them.
*   **Automation:**
    1.  The user enables auto-patching for their WordPress site by providing SSH credentials.
    2.  They configure notifications to be alerted of any changes or issues.

### 2.2. Non-Technical but Comfortable User

**Journey:** This user wants a guided experience to improve their website's security and reliability.

*   **Onboarding:**
    1.  The user signs up and is prompted to add their domain.
    2.  The platform scans the domain and presents a simplified health check with clear explanations of any issues found.
    3.  The user is guided through the process of changing their NS records to delegate DNS management to SafeOrbit.
*   **Configuration:**
    1.  The platform presents a series of recommended actions, such as "Enable HTTPS" and "Protect your email."
    2.  The user clicks on each recommendation and is shown a simple explanation of what will be done.
    3.  They approve the changes, and the platform takes care of the technical implementation.

### 2.3. "I Don’t Know How I Got Here" User

**Journey:** This user is looking for a simple, one-click solution to fix any problems with their website.

*   **Onboarding:**
    1.  The user lands on a simplified version of the dashboard, possibly from a link sent by their web developer.
    2.  The dashboard shows a large, color-coded status indicator (green, yellow, or red) for their website.
*   **Configuration:**
    1.  If the status is yellow or red, a single, prominent button is displayed: "Fix Safely."
    2.  The user clicks the button, and the platform automatically applies a set of safe, pre-approved fixes.
    3.  The user is shown a confirmation message that their site is now secure and healthy.

## 3. Wireframe-Level Descriptions

*   **Landing & Onboarding:** A clean, simple page with a single input field for the user to enter their domain name.
*   **Domain Dashboard:** A high-level overview of the domain's health, with clear status indicators for DNS, HTTPS, Email, and Security.
*   **DNS Management Panel:** A table-based view of all DNS records, with options to add, edit, and delete records. Templates and a propagation status visualizer are also present.
*   **Email Authentication Panel:** A guided wizard to help users configure SPF, DKIM, and DMARC, with a "spoof-resistance score" and clear explanations.
*   **Security Settings & Profiles:** A simple interface to select a security profile (Relaxed, Balanced, Strict) and enable/disable specific security features.
*   **Scan Results & History:** A chronological log of all scans and changes made to the domain, with the ability to view details and rollback changes.
