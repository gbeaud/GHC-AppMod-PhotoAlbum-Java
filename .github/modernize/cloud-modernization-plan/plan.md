# Modernization Plan: Cloud Readiness, Azure Infrastructure & Deployment

**Project**: Photo Album (Java Spring Boot)

---

## Technical Framework

- **Language**: Java 25
- **Framework**: Spring Boot 4.0.0
- **Build Tool**: Maven
- **Database**: Oracle Database 21c Express Edition
- **Key Dependencies**: Spring Data JPA, Hibernate, Thymeleaf, Oracle JDBC (ojdbc8), Spring Boot Validation, Spring Security

> **Note**: This application was previously assessed and has since been upgraded to Spring Boot 4.0.0 on
> Java 25. Any upgrade recommendations from the prior assessment report are superseded and out of scope
> for this plan. This plan focuses only on remaining cloud readiness issues, Azure infrastructure
> provisioning, and deployment.

---

## Overview

> This modernization prepares the Photo Album application for Azure by resolving its remaining cloud
> readiness gaps, provisioning the required Azure infrastructure, and deploying the application to Azure.
> The application currently stores all data in an on-premises Oracle Database container and authenticates
> to it with a static username/password pair supplied through environment variables. The new architecture
> will:
>
> - Replace the Oracle Database dependency with Azure Database for PostgreSQL Flexible Server, migrating
>   JDBC drivers, SQL dialect, and schema so the application no longer depends on a proprietary,
>   self-hosted database engine.
> - Remove the static database password entirely by authenticating to PostgreSQL with Microsoft Entra ID
>   / managed identity, closing a cloud-readiness/security gap left over from the on-premises setup.
> - Remediate known CVEs in project dependencies so the application is deployed with a clean security
>   posture.
> - Provision the Azure infrastructure (compute, database, networking, identity) required to run the
>   application using Infrastructure as Code (Bicep).
> - Containerize and deploy the application to Azure Container Apps, replacing local Docker Compose
>   orchestration with a managed, scalable Azure runtime.
>
> The migration follows a phased approach: baseline capture and infrastructure provisioning run in
> parallel with the database migration, followed by integration verification, security remediation, and
> finally deployment.

---

## Migration Impact Summary

| Application | Original Service | New Azure Service | Authentication | Comments |
|-------------|------------------|-------------------|-----------------|----------|
| Photo Album | Oracle Database 21c XE (self-hosted, password) | Azure Database for PostgreSQL Flexible Server | Managed Identity | Migrates BLOB/schema, removes hardcoded password |
| Photo Album | Docker Compose (local host) | Azure Container Apps | Managed Identity | Containerized deployment, provisioned via Bicep |

---

## Open Questions & Questionnaire

- [x] Q: Should the plan include environment/infrastructure provisioning? → A: Yes — provision new
      infrastructure (explicitly requested by user).
- [x] Q: Should the plan include integration testing to verify migrated services? → A: Yes — Real mode,
      using the infrastructure provisioned by this plan (default when infrastructure is provisioned).
- [x] Q: Should the plan include a security scan and CVE remediation task? → A: Yes — include
      security/CVE remediation (default).
- [x] Q: Which Azure deployment target should the plan use? → A: Azure Container Apps (default;
      includes containerization, no separate containerization task needed).
- [x] Q: Should a Java/framework upgrade task be included? → A: No — the application has already been
      upgraded to Spring Boot 4.0.0 / Java 25; upgrade recommendations from the prior assessment are
      ignored per the user's request.
- [x] Q: Should the OracleDB be migrated? → A: Yes — migrate to PostgreSQL, per explicit user request.

---

## Proposed Architecture

```
                 ┌────────────────────────────┐
                 │   Azure Container Apps      │
                 │   (Photo Album app image)   │  ── Managed Identity ──┐
                 └───────────┬────────────────┘                        │
                             │ HTTPS                                    │
                             ▼                                          ▼
                 ┌────────────────────────────┐          ┌───────────────────────────────┐
                 │  Azure Container Registry   │          │  Azure Database for PostgreSQL │
                 │  (application image)        │          │  Flexible Server               │
                 └────────────────────────────┘          └───────────────────────────────┘
                             │
                             ▼
                 ┌────────────────────────────┐
                 │  Log Analytics / App        │
                 │  Insights (Container Apps   │
                 │  environment diagnostics)   │
                 └────────────────────────────┘
```

---

## Azure Resource List

| Resource Type | Resource Name | SKU | Est. Monthly Cost | Purpose |
|---------------|---------------|-----|--------------------|---------|
| Container Apps Environment | cae-photoalbum | Consumption | ~$0 (usage-based) | Hosting environment for the app |
| Container App | ca-photoalbum | 0.5 vCPU / 1 GiB | ~$15-30/mo | Runs the Photo Album Spring Boot app |
| Container Registry | acrphotoalbum | Basic | ~$5/mo | Stores the application container image |
| Azure Database for PostgreSQL Flexible Server | psql-photoalbum | Burstable B1ms | ~$15-25/mo | Replaces Oracle DB for photo/app data |
| User-Assigned Managed Identity | id-photoalbum | N/A | Free | Passwordless auth to PostgreSQL & ACR |
| Log Analytics Workspace | log-photoalbum | Pay-as-you-go | ~$5-10/mo | Diagnostics for Container Apps |

> **Note**: Estimated costs are based on Azure retail prices and serve only as a rough estimation. Actual
> costs may differ due to enterprise agreements, reservations, region, or actual consumption.

---

## Task Summary

| # | Task ID | Type | Description |
|---|---------|------|-------------|
| 1 | 000-setupBaseline | setupBaseline | Capture pre-migration baseline (test cases, testdata, infra-decision-table) from `src/main` |
| 2 | 001-infrastructure-bicep-generation | infrastructure | Generate & provision Bicep IaC for Container Apps, PostgreSQL, ACR, managed identity |
| 3 | 002-transform-migration-oracle-to-postgresql | transform | Migrate Oracle DB to Azure Database for PostgreSQL with managed identity authentication |
| 4 | 003-integrationTest | integrationTest | Verify the PostgreSQL migration against provisioned Azure infrastructure |
| 5 | 004-security-cve-remediation | security | Scan and remediate CVEs in project dependencies |
| 6 | 005-deployment-azure-container-apps | deployment | Containerize and deploy the application to Azure Container Apps |

See `.metadata/tasks.json` for full task definitions, dependencies, and success criteria.
