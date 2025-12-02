# API Specifications: SafeOrbit

## 1. Internal APIs (Frontend to Backend)

The internal API will be a combination of RESTful endpoints and RPC calls, implemented as Supabase Edge Functions. The frontend will interact with these endpoints to fetch data and trigger actions.

### 1.1. Endpoints

*   **`POST /api/scan`**: Initiates a new scan for a domain.
    *   **Request Body:** `{ "domain": "example.com" }`
    *   **Response:** `{ "jobId": "12345" }`
*   **`GET /api/scan/{jobId}`**: Retrieves the status and results of a scan.
    *   **Response:** `{ "status": "completed", "results": { ... } }`
*   **`GET /api/domains/{domainId}/dns`**: Fetches the DNS records for a domain.
    *   **Response:** `[ { "type": "A", "name": "@", "value": "1.2.3.4" }, ... ]`
*   **`POST /api/domains/{domainId}/dns`**: Creates a new DNS record.
    *   **Request Body:** `{ "type": "A", "name": "@", "value": "1.2.3.4" }`
*   **`PUT /api/domains/{domainId}/dns/{recordId}`**: Updates an existing DNS record.
*   **`DELETE /api/domains/{domainId}/dns/{recordId}`**: Deletes a DNS record.

## 2. Provider Integration APIs

These are the APIs of the external services that SafeOrbit will integrate with. The platform will have a generic interface for each type of provider (e.g., DNS, WAF), with specific implementations for each service.

### 2.1. Cloudflare API

*   **Endpoints Used:**
    *   `/zones`
    *   `/zones/{zone_id}/dns_records`
    *   `/zones/{zone_id}/firewall/rules`
*   **Configuration Mapping:** The platform will map its internal data models for DNS records and firewall rules to the corresponding Cloudflare API objects.

### 2.2. Bunny API

*   **Endpoints Used:**
    *   `/dnszone`
    *   `/dnszone/{id}`
*   **Configuration Mapping:** Similar to the Cloudflare integration, the platform will map its internal models to the Bunny API objects.
