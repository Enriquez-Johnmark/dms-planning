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

## Anything else?

Anything about Officetul that a new module should know about, or must not break?

>
