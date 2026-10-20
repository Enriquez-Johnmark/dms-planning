# Officetul Technical Questionnaire

**Purpose:** the document management module will be built inside Officetul. The repository isn't available until design is done, so these answers let us design the module to fit Officetul instead of guessing. A design review against the real code follows once access is granted (decision D-013).

**From:** the project owner. **To:** whoever builds or maintains Officetul. **How your answers will be used:** they go into the design documents in this planning repository. No code or credentials are needed. Please don't include passwords, keys or client data.

## How to answer

About 20–30 minutes. Short answers are fine. "I don't know" is useful too: mark anything you're unsure of rather than skipping it.

## Multi-tenancy

### 1. How does Officetul keep each client's data separate?
For example: one database with a `tenant_id` (or `company_id`) column on every table, a separate database per client, or a package such as `stancl/tenancy` or `spatie/laravel-multitenancy`.

_Why this matters: the DMS must isolate files, search results and background jobs in exactly the same way, or one client could see another's documents._

>

### 2. How does Officetul know which client a request belongs to?
For example: subdomain (`client.officetul.com`), custom domain, URL path, or the logged-in user's company.

>

### 3. Can one user belong to more than one client company?

>

## Application structure

### 4. Which versions are in use: PHP, Laravel, Inertia, React, MySQL (or MariaDB), Node?

>

### 5. How are apps such as Accounting and Inventory organised in the code?
For example: folders under `app/`, a modules package such as `nwidart/laravel-modules`, or separate Composer packages. How is an app added to the main menu?

>

### 6. Do any Officetul apps already let users attach files to records (e.g. a receipt on an expense)? If yes, where are those files stored?

>

## Users and permissions

### 7. How do users log in?
For example: Breeze, Jetstream, Fortify, custom, or SSO. Is two-factor authentication available?

>

### 8. How are roles and permissions managed?
For example: `spatie/laravel-permission`, a custom table, or per-app permissions. Can a client admin create their own roles?

_Why this matters: document access (who can see which folder) has to plug into the same permission system._

>

### 9. Is there an existing activity or audit log?
For example: `spatie/laravel-activitylog` or a custom log. What does it record?

>

## Infrastructure

### 10. Where are uploaded files stored today?
For example: the server's local disk, Amazon S3, DigitalOcean Spaces, or another S3-compatible service. Are files separated by client?

>

### 11. Are background jobs and queues used?
Which driver (database, Redis)? Are workers kept running (Supervisor, Horizon)? Is the scheduler (`schedule:run`) set up?

_Why this matters: OCR and virus scanning must run in the background, not while the user waits._

>

### 12. Where is Officetul hosted, and on how many servers?
Provider, rough server size (CPU, RAM, disk), operating system, and who manages it.

>

### 13. How are the database and files backed up, how often, and where to? Has anyone ever restored from a backup to test it?

>

### 14. Does Officetul send or receive email? Which service (SMTP provider, Mailgun, SES, Postmark…)?

>

### 15. Is any search engine used (MySQL FULLTEXT, Laravel Scout, Meilisearch, Elasticsearch), or only normal database filters?

>

## Scale and process

### 16. Roughly how many client companies and users are on Officetul today? How many users does the largest client have?

>

### 17. How is a new version deployed? Is there a staging environment? Are there automated tests?

>

## Security details (added after Phase 8)

These follow up questions 1, 8 and 9. "I don't know" is fine.

### 18. If code runs without a current client company (a background job, a scheduled command), what happens to database queries?
For example: they return every company's rows, they return nothing, or they stop with an error.

_Why this matters: the Documents app will stop with an error in that case (decision D-052), and we need to know whether the rest of Officetul behaves the same way._

>

### 19. How does Officetul decide who may open a particular record, such as an expense or a project?
For example: any user with the right role, only the record's owner, or by branch or department.

_Why this matters: a document linked to a record can be shown to people who may open that record (D-053)._

>

### 20. Is there a "log in as user" (impersonation) feature, or do staff ever query the production database directly?

_Why this matters: Officetul staff will see client documents only with the client's approval (D-057)._

>

### 21. Which database user does the app connect with? Is it an admin or root user? Do migrations run with the same user? Is the database on Amazon RDS or self-managed?

_Why this matters: the activity log will be insert-only, which needs a separate user for migrations (D-061)._

>

### 22. Are record IDs in URLs plain numbers (`/expenses/881`) or random IDs (UUID or ULID)?

>

### 23. Is anything cached in front of logged-in pages (a CDN, a reverse proxy, page caching)? Are cache keys separated per company?

>

### 24. Which PHP version is in use, and does Officetul use Laravel Octane or other long-running workers?

>

## Anything else?

Anything about Officetul that a new module should know about, or must not break?

>
