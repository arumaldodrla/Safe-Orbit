# Data Model & Database Schema: SafeOrbit

## 1. Introduction

This document provides a comprehensive specification of the SafeOrbit database schema. The data model is designed to support multi-tenancy, audit trails, and the full lifecycle of domain scanning and security management.

## 2. Entity Relationship Diagram

```
┌─────────────┐
│    Users    │
└──────┬──────┘
       │
       │ 1:N
       │
┌──────▼──────────────┐      ┌──────────────────────┐
│     Domains         │◄─────┤ ProviderConnections  │
└──────┬──────────────┘      └──────────────────────┘
       │
       │ 1:N
       │
       ├──────────────┬──────────────┬──────────────┬──────────────┐
       │              │              │              │              │
┌──────▼──────┐ ┌────▼─────┐ ┌──────▼──────┐ ┌─────▼──────┐ ┌────▼─────────┐
│ DNSRecords  │ │ ScanJobs │ │  Findings   │ │  Security  │ │  ChangeLogs  │
│             │ │          │ │             │ │  Profiles  │ │              │
└─────────────┘ └────┬─────┘ └─────────────┘ └────────────┘ └──────────────┘
                     │
                     │ 1:N
                     │
               ┌─────▼─────┐
               │ Findings  │
               └───────────┘
```

## 3. Core Entities

### 3.1. Users

The `users` table stores all user accounts and authentication information.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `openId` | VARCHAR(64) | NOT NULL, UNIQUE | Manus OAuth identifier |
| `name` | TEXT | | User's display name |
| `email` | VARCHAR(320) | | User's email address |
| `loginMethod` | VARCHAR(64) | | Authentication method (e.g., "manus") |
| `role` | ENUM('user', 'admin') | NOT NULL, DEFAULT 'user' | User role for access control |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Account creation timestamp |
| `updatedAt` | TIMESTAMP | NOT NULL, DEFAULT NOW(), ON UPDATE NOW() | Last update timestamp |
| `lastSignedIn` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last successful login |

**Indexes:**
- PRIMARY KEY on `id`
- UNIQUE INDEX on `openId`

**Multi-Tenancy:** Each user is isolated by their `id`. All domain-related data references `userId` to enforce data separation.

### 3.2. Domains

The `domains` table represents websites managed by SafeOrbit.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `userId` | INT | NOT NULL, FOREIGN KEY → users.id | Owner of this domain |
| `domain` | VARCHAR(255) | NOT NULL, UNIQUE | Fully qualified domain name |
| `status` | ENUM('active', 'pending', 'error', 'paused') | NOT NULL, DEFAULT 'pending' | Current domain status |
| `lastScanAt` | TIMESTAMP | | Timestamp of most recent scan |
| `healthScore` | INT | | Overall health score (0-100) |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Domain added timestamp |
| `updatedAt` | TIMESTAMP | NOT NULL, DEFAULT NOW(), ON UPDATE NOW() | Last modification timestamp |

**Indexes:**
- PRIMARY KEY on `id`
- UNIQUE INDEX on `domain`
- INDEX on `userId` for efficient user domain lookups

**Business Rules:**
- A domain can only belong to one user
- Health scores are calculated based on findings from the most recent scan
- Status transitions: `pending` → `active` (after first successful scan) or `error` (if scan fails)

### 3.3. ProviderConnections

The `providerConnections` table stores API credentials for external DNS and security providers.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `userId` | INT | NOT NULL, FOREIGN KEY → users.id | Owner of this connection |
| `domainId` | INT | FOREIGN KEY → domains.id | Optional: specific domain this connection applies to |
| `provider` | ENUM('cloudflare', 'bunny', 'other') | NOT NULL | Provider type |
| `apiKey` | TEXT | NOT NULL | Encrypted API key or token |
| `apiEmail` | VARCHAR(320) | | Email associated with the API account (for Cloudflare) |
| `zoneId` | VARCHAR(255) | | Provider-specific zone identifier |
| `status` | ENUM('active', 'error', 'revoked') | NOT NULL, DEFAULT 'active' | Connection status |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Connection created timestamp |
| `updatedAt` | TIMESTAMP | NOT NULL, DEFAULT NOW(), ON UPDATE NOW() | Last update timestamp |

**Security Considerations:**
- API keys must be encrypted at rest using AES-256
- Keys should never be exposed in logs or API responses
- Implement key rotation policies

### 3.4. DNSRecords

The `dnsRecords` table stores snapshots of DNS records discovered during scans.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `domainId` | INT | NOT NULL, FOREIGN KEY → domains.id | Associated domain |
| `recordType` | VARCHAR(10) | NOT NULL | DNS record type (A, AAAA, CNAME, MX, TXT, NS, CAA, etc.) |
| `name` | VARCHAR(255) | NOT NULL | Record name (e.g., "@", "www", "_dmarc") |
| `value` | TEXT | NOT NULL | Record value |
| `ttl` | INT | | Time to live in seconds |
| `priority` | INT | | Priority (for MX records) |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record discovered timestamp |
| `updatedAt` | TIMESTAMP | NOT NULL, DEFAULT NOW(), ON UPDATE NOW() | Last seen timestamp |

**Indexes:**
- PRIMARY KEY on `id`
- INDEX on `domainId` for efficient domain record lookups
- COMPOSITE INDEX on (`domainId`, `recordType`) for filtered queries

**Data Retention:** Old DNS records should be archived or deleted after a configurable period (e.g., 90 days) to prevent table bloat.

### 3.5. ScanJobs

The `scanJobs` table tracks all scanning operations.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `domainId` | INT | NOT NULL, FOREIGN KEY → domains.id | Domain being scanned |
| `scanType` | ENUM('full', 'dns', 'https', 'email', 'security') | NOT NULL | Type of scan |
| `status` | ENUM('pending', 'running', 'completed', 'failed') | NOT NULL, DEFAULT 'pending' | Current job status |
| `startedAt` | TIMESTAMP | | Scan start time |
| `completedAt` | TIMESTAMP | | Scan completion time |
| `errorMessage` | TEXT | | Error details if status is 'failed' |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Job created timestamp |

**Indexes:**
- PRIMARY KEY on `id`
- INDEX on `domainId` for job history retrieval
- INDEX on `status` for monitoring pending/running jobs

**Job Lifecycle:**
1. `pending` → Job created, waiting to execute
2. `running` → Job is actively scanning
3. `completed` or `failed` → Terminal states

### 3.6. Findings

The `findings` table stores security issues and recommendations discovered during scans.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `scanJobId` | INT | NOT NULL, FOREIGN KEY → scanJobs.id | Scan that discovered this finding |
| `domainId` | INT | NOT NULL, FOREIGN KEY → domains.id | Affected domain |
| `category` | ENUM('dns', 'https', 'email', 'security', 'performance') | NOT NULL | Finding category |
| `severity` | ENUM('critical', 'high', 'medium', 'low', 'info') | NOT NULL | Severity level |
| `title` | VARCHAR(255) | NOT NULL | Short finding title |
| `description` | TEXT | NOT NULL | Detailed explanation |
| `recommendation` | TEXT | | Suggested remediation steps |
| `autoFixable` | INT | NOT NULL, DEFAULT 0 | Boolean: can this be auto-fixed? (0 or 1) |
| `resolved` | INT | NOT NULL, DEFAULT 0 | Boolean: has this been resolved? (0 or 1) |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Finding discovered timestamp |

**Indexes:**
- PRIMARY KEY on `id`
- INDEX on `domainId` for domain-specific findings
- INDEX on `resolved` for filtering unresolved issues
- COMPOSITE INDEX on (`domainId`, `severity`, `resolved`) for dashboard queries

**Severity Mapping:**
- `critical`: Immediate action required (e.g., no HTTPS, exposed admin panel)
- `high`: Significant security risk (e.g., missing DMARC, weak TLS)
- `medium`: Moderate risk (e.g., missing security headers)
- `low`: Minor improvements (e.g., suboptimal TTL values)
- `info`: Informational only (e.g., IPv6 not configured)

### 3.7. SecurityProfiles

The `securityProfiles` table defines security configurations for each domain.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `domainId` | INT | NOT NULL, FOREIGN KEY → domains.id | Associated domain |
| `profileType` | ENUM('relaxed', 'balanced', 'strict') | NOT NULL, DEFAULT 'balanced' | Security profile level |
| `wafEnabled` | INT | NOT NULL, DEFAULT 0 | Boolean: WAF enabled (0 or 1) |
| `rateLimitEnabled` | INT | NOT NULL, DEFAULT 0 | Boolean: rate limiting enabled |
| `httpsForceEnabled` | INT | NOT NULL, DEFAULT 1 | Boolean: force HTTPS redirect |
| `securityHeadersEnabled` | INT | NOT NULL, DEFAULT 1 | Boolean: security headers enabled |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Profile created timestamp |
| `updatedAt` | TIMESTAMP | NOT NULL, DEFAULT NOW(), ON UPDATE NOW() | Last modification timestamp |

**Profile Definitions:**
- **Relaxed**: Minimal security, suitable for development or internal sites
- **Balanced**: Recommended default with essential protections
- **Strict**: Maximum security, may break some legacy applications

### 3.8. ChangeLogs

The `changeLogs` table provides a complete audit trail of all modifications.

| Field | Type | Constraints | Description |
|:------|:-----|:------------|:------------|
| `id` | INT | PRIMARY KEY, AUTO_INCREMENT | Surrogate primary key |
| `userId` | INT | NOT NULL, FOREIGN KEY → users.id | User who made the change |
| `domainId` | INT | NOT NULL, FOREIGN KEY → domains.id | Affected domain |
| `action` | VARCHAR(100) | NOT NULL | Action performed (e.g., "dns_record_added") |
| `resourceType` | VARCHAR(50) | NOT NULL | Type of resource modified |
| `resourceId` | INT | | ID of the modified resource |
| `changeDetails` | TEXT | | JSON string with before/after values |
| `createdAt` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Change timestamp |

**Indexes:**
- PRIMARY KEY on `id`
- INDEX on `domainId` for domain change history
- INDEX on `createdAt` for chronological queries

**Use Cases:**
- Rollback functionality (revert to previous state)
- Compliance and auditing
- Debugging user-reported issues

## 4. Data Access Patterns

### 4.1. Common Queries

**Retrieve all domains for a user:**
```sql
SELECT * FROM domains WHERE userId = ? ORDER BY createdAt DESC;
```

**Get latest scan results for a domain:**
```sql
SELECT f.* FROM findings f
JOIN scanJobs sj ON f.scanJobId = sj.id
WHERE f.domainId = ? AND sj.status = 'completed'
ORDER BY f.createdAt DESC;
```

**Find unresolved critical findings:**
```sql
SELECT * FROM findings
WHERE domainId = ? AND resolved = 0 AND severity = 'critical'
ORDER BY createdAt DESC;
```

### 4.2. Performance Considerations

- Use connection pooling for database access
- Implement query result caching for frequently accessed data (e.g., domain lists)
- Archive old scan jobs and findings to separate tables after 90 days
- Use read replicas for reporting and analytics queries

## 5. Migration Strategy

All schema changes must be managed through Drizzle ORM migrations:

1. Update `drizzle/schema.ts` with the new schema
2. Run `pnpm db:push` to generate and apply migrations
3. Test migrations in a staging environment before production deployment
4. Document breaking changes in migration notes

## 6. Backup and Recovery

- **Automated Backups**: Daily full backups with point-in-time recovery
- **Retention Policy**: Keep daily backups for 30 days, weekly for 90 days
- **Disaster Recovery**: Test restore procedures quarterly
- **Critical Tables**: Users, Domains, ProviderConnections (contains encrypted credentials)

---

**Document Version**: 1.0  
**Last Updated**: December 2024  
**Maintained By**: Manus AI
