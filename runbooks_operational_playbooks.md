# Runbooks & Operational Playbooks: SafeOrbit

## 1. Routine Tasks

### 1.1. Onboarding a New Customer Domain

1.  **Symptom:** A new user signs up and adds a domain.
2.  **Triage:**
    *   Verify that the domain is valid and not already in the system.
3.  **Action:**
    *   Initiate the initial domain scan.
    *   Guide the user through the process of either delegating their NS records or connecting their DNS provider via API.

### 1.2. Rotating Provider API Tokens

1.  **Symptom:** A provider API token is nearing its expiration date or has been compromised.
2.  **Triage:**
    *   Identify all domains that are using the token.
3.  **Action:**
    *   Generate a new token in the provider's dashboard.
    *   Update the token in the SafeOrbit database.
    *   Verify that the connection to the provider is still working.

### 1.3. Responding to Failed Scans

1.  **Symptom:** A scheduled scan fails to complete.
2.  **Triage:**
    *   Check the scan logs for error messages.
    *   Determine if the failure was due to a transient issue (e.g., a network error) or a persistent problem (e.g., an invalid domain).
3.  **Action:**
    *   If the issue is transient, retry the scan.
    *   If the issue is persistent, create a notification for the user or a ticket for the support team.

## 2. Incident Responses

### 2.1. DNS Changes Broke a Site

1.  **Symptom:** A user reports that their site is down after making a DNS change through SafeOrbit.
2.  **Triage:**
    *   Immediately review the change log for the domain to identify the recent DNS change.
    *   Verify that the site is indeed down and that the DNS change is the likely cause.
3.  **Action:**
    *   Use the rollback feature to revert the DNS change.
    *   Communicate with the user to let them know that the issue is being addressed.
    *   Investigate the root cause of the problem to prevent it from happening again.

### 2.2. Email is Disrupted Due to SPF/DMARC Change

1.  **Symptom:** A user reports that they are not receiving emails after a change to their SPF or DMARC records.
2.  **Triage:**
    *   Review the change log to identify the recent SPF/DMARC change.
    *   Use an email testing tool to verify that email is not being delivered.
3.  **Action:**
    *   Rollback the change to the SPF/DMARC records.
    *   Work with the user to correctly configure their email authentication settings.

### 2.3. Cloudflare/Bunny API is Unavailable

1.  **Symptom:** The platform is unable to connect to the Cloudflare or Bunny API.
2.  **Triage:**
    *   Check the status page for the provider to see if there is a known outage.
    *   Verify that the platform's API credentials are still valid.
3.  **Action:**
    *   If there is a provider outage, display a notification to users who are affected.
    *   If the issue is with the platform's credentials, update them.
    *   The platform should be designed to degrade gracefully in the event of a provider outage, with read-only access to DNS records and other data.
