# Clinic & Salon ERP

Israel-first clinic and salon ERP. Specification **1.17-draft** is the source of truth: one .NET solution, Azure locked, Israel launch. This organization does not split the product across mobile, data, infrastructure, or security repositories.

The earlier profile names `clinix`, `clinix-mobile`, `clinix-data`, `clinix-afterarc`, `clinix-infra`, `clinix-security`, `clinix-pulse`, and `clinix-ops` are retired. They are not product repositories.

| Repo | Role |
|---|---|
| [nano-erp-devices](https://github.com/clinix-erp/nano-erp-devices) | TypeSpec contracts for the virtualized clinic devices and the Defined Access Protocol used to stream, control, and inject hardware events into the stakeholder emulator. |
| [nano-erp](https://github.com/clinix-erp/nano-erp) | Private .NET 10 / Aspire product solution: API modules, web, portal, shared UI, MAUI Android, tenant WhatsApp `ingestion/`, `registry/`, Aspire deployment under `eng/`, and `compliance/`. |

## Where the pipelines live

These are separate systems. A WhatsApp export never enters Data Factory, and a Ministry of Health product file never enters `ingestion/`.

**Tenant WhatsApp ingestion** is `ingestion/` inside [nano-erp](https://github.com/clinix-erp/nano-erp) (specification §7.9).

- Input is WhatsApp history files, live WhatsApp capture, and expense documents for one clinic tenant.
- The service writes `ingestiondb`, shows the review grid in the web app, and publishes IDs-only events on Service Bus. The Integrations module applies accepted rows to the tenant database.
- This is personal data, gated by the communications-ingestion DPIA. Foundry calls from this path use EU DataZone.
- It runs as an Aspire Container App in the application resource group. It is not a data-integration resource group. `rg-text-to-struct` was an old test bed for this code before the merge into `ingestion/`.

**Jurisdiction data factory** is Azure Data Factory in the resource group `rg-erp-data-factory`, not a GitHub repository.

- It registers public product and regulatory sources from more than one jurisdiction, routes each source, runs ETL, and loads `registrydb`.
- Launch sources are the Israeli registries and Ministry of Health notifications, plus openFDA as the agency fallback. Later jurisdictions are additional sources in this factory, not new copies of the WhatsApp service.
- The load target is platform-global public product facts, with no tenant id. Staff and the tablet read the loaded snapshot through `registry-api`. End users never open Data Factory.
- `registry/` inside `nano-erp` is the read API, the nightly orchestrator, and the ops TUI over the database the factory fills. It is not the ETL tool.
- The factory resource group is persistent. Tearing down the staging app does not delete its source definitions, pipelines, or triggers.

**Aspire infrastructure** is `eng/` inside `nano-erp` (`eng/deploy.cs`, `eng/teardown.cs`, `eng/check-publish.cs`, and Aspire Bicep publish). There is no separate infrastructure repository. Azure is locked: Sweden Central for staging, compute, and AI until Container Apps is available in Israel Central; Israel Central is the production data-plane residency target.

Security policies and threat models live in `compliance/` inside `nano-erp`. Company operations stay outside product repositories. Tier 2 post-operative monitoring is a specification design note and is not a Phase 1 repository.
