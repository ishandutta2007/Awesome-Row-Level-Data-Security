# Awesome-Row-Level-Data-Security

# Top Row-Level Data Security Ecosystem

**Curated List of SaaS Products & Open-Source GitHub Projects**  
*Focused on Row Access Policies, Attribute-Based Controls & Self-Hosted Data Governance*  
**Last updated: November 2026**

This repository tracks notable **SaaS platforms** and **open-source projects** for **Row-Level Data Security**. These tools enable organizations to control *which rows* a user can see within a shared table—enforcing multi-tenant isolation, regional data residency, and least-privilege access without duplicating datasets or building separate views for every audience.

**Examples** include Snowflake Row-Level Security, Immuta, Privacera, Satori, Cyral, ALTR, Protegrity, SecuPi, Microsoft Purview, and Databricks Unity Catalog (the category leaders).

**Open-source emphasis**: Row-Level Data Security is a domain where open-source tooling is **genuinely emerging but still maturing at the governance layer**. The strongest active project is **AccessFlow**, a full query proxy that sits between users and databases to enforce row-level security and column masking policies before queries execute—not after, and not via client-side enforcement . For warehouse-native patterns, **Snowflake Row Access Policies** and **Databricks Unity Catalog** provide SQL-based predicates (though they are platform features, not standalone products) . For PostgreSQL developers, **rlsalchemy** brings declarative RLS policy management to SQLAlchemy and Alembic, making tenant isolation a schema-level concern rather than an application-code afterthought . The critical caveat: **no open-source project matches Immuta's or Privacera's breadth of platform connectors and centralized policy authoring**—those remain commercial differentiators . And row-level security carries real performance costs: Snowflake's row access policies add 10–30% overhead on large tables unless the policy column aligns with the clustering key .

Contributions welcome! Open a PR to add/update entries. Keep descriptions factual and link to official sites.

## Table of Contents
- [SaaS/Hosted Platforms](#saas-products)
- [Open-Source GitHub Projects](#open-source-github-projects)
- [How to Contribute](#how-to-contribute)
- [Disclaimer](#disclaimer)

## SaaS/Hosted Platforms

- **[Snowflake Row-Level Security](https://docs.snowflake.com/en/user-guide/security-row-intro)**  
  Native **Row Access Policies** that filter rows based on role or session context; the policy is a SQL expression returning boolean—TRUE makes the row visible, FALSE hides it entirely . The multi-tenant pattern uses session variables (e.g., `department_id`) so each department sees only its own data from a shared table. **Critical caveat**: policies add a predicate to every query; on 100M+ row tables, expect **10–30% overhead** unless the policy column aligns with the clustering key . Masking policies (column-level) are a separate feature and cannot reference other tables—only context functions like `CURRENT_ROLE()` .

- **[Databricks Unity Catalog Row Filters](https://docs.databricks.com/aws/en/data-governance/unity-catalog/filters-and-masks)**  
  Table-level **row filters** bound via `ALTER TABLE ... SET ROW FILTER` and managed by the table owner; column masks similarly attached per column . **Recommended path for scale**: **ABAC policies** attach at catalog or schema level via governed tags, applying automatically to new tables as they are tagged and preventing table owners from overriding . **Performance guidance**: use simple SQL expressions, limit distinct masks on large tables, reduce UDF arguments, and prefer deterministic expressions that cannot throw errors (e.g., `try_divide`) to allow optimizer push-down .

- **[Immuta](https://www.immuta.com/)**  
  Data access governance platform that **enforces policies directly**—queries return filtered/masked results based on policy logic, not just posture reporting . Positioned for organizations needing centralized policy authoring across multiple data platforms, with attribute-based controls and row-level filtering.

- **[Privacera](https://privacera.com/)**  
  Data security and governance platform with **Row-Level Security (RLS)** and **Column-Level Security (CLS)** via Apache Ranger API integration . Supports 50+ connectors across structured and semi-structured sources; offers cloud-native and self-managed deployments. Includes **Privacera Encryption Gateway** for dynamic data masking and format-preserving encryption . **Best for organizations wanting a commercial platform with broad connector coverage** and Ranger ecosystem alignment.

- **[Satori](https://www.satori.com/)**  
  Data access governance platform (acquired by Varonis) that enforces policies at query time; positioned for cloud data warehouses and databases . **Note**: as part of Varonis, verify current product direction and standalone availability.

- **[ALTR](https://www.altra.io/)**  
  Data access governance with **tokenization** as a key differentiator—stores surrogate values by design, placing data in a second system . Enforces row-level and masking policies at query time. **Best for organizations needing tokenization alongside RLS**.

- **[Cyral](https://www.cyral.com/)**  
  **Acquired by Varonis in March 2025**—any 2026 shortlist carrying Cyral as a standalone vendor is stale . Verify current product status before evaluation.

- **[Protegrity](https://www.protegrity.com/)**  
  Data security platform with fine-grained access control, encryption, and masking; positioned for enterprise data protection across databases and analytics platforms.

- **[SecuPi](https://www.secupi.com/)**  
  Data security and privacy platform with policy enforcement for row-level access and masking; positioned for regulated industries.

- **[Microsoft Purview](https://learn.microsoft.com/en-us/purview/purview)**  
  Microsoft's data governance platform; **Purview itself is not a row-level security enforcement engine**—it provides discovery, classification, and governance. RLS enforcement happens in the underlying platform (e.g., Azure SQL, Power BI) with Purview providing visibility and policy guidance . **Best for Microsoft-centric organizations** wanting governance visibility alongside platform-native RLS.

## Open-Source GitHub Projects

### Query Proxy Enforcement (Row-Level Security)

- **[AccessFlow](https://github.com/bablsoft/accessflow)**  
  The most complete open-source **database and API access governance platform** with **row-level security and column masking enforced by the proxy, not the client** . Features: **full query proxy** sitting between users and databases (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, MongoDB, Couchbase, Redis, Cassandra, ScyllaDB, Elasticsearch, OpenSearch, DynamoDB, Neo4j, and any JDBC-compatible engine); **AI-powered query analysis** before human review (OpenAI, Anthropic, Ollama, or Hugging Face); **multi-stage approval chains**; **row-security and masking policies**; **tamper-evident audit log** (HMAC-chained, INSERT-only); just-in-time access, break-glass, and attestation campaigns; **API access governance** (REST, SOAP, GraphQL, gRPC) with response masking; and **deployment approval governance** for CI/CD pipelines . Terraform provider available for declarative policy management. **Best for organizations wanting a self-hosted, proxy-based enforcement point** that works across heterogeneous databases—but it is a proxy, so availability and residency requirements sit inside your data path .

### PostgreSQL-Native RLS (Declarative Schema)

- **[rlsalchemy](https://pypi.org/project/rlsalchemy/)**  
  Open-source **declarative PostgreSQL and CockroachDB row-level security** for SQLAlchemy 2.1 and Alembic 1.18 . Models own their policy expressions via `__rls__`; a typed context model derives every setting name, SQL cast, and prefix from one class; Alembic autogenerate compares the declaration with the live database and writes reversible operations for complete table state . Features: **crud()** helper producing separate select/insert/update/delete policies; **verification API** to check live database against declarations without generating a migration; **drift detection** with typed `AlterRLSOp` carrying before/after states . **Pre-release caveat**: earlier versions were marked as pre-release/not stable for production —verify current stability before deploying. **Best for PostgreSQL teams** wanting tenant isolation as a schema-level concern with migration support—but it is PostgreSQL/CockroachDB-specific and does not provide cross-platform governance.

### Additional Strong Open-Source Options

- **Apache Ranger** — Open-source security framework for Hadoop ecosystems with **row-level filtering and column masking** policies via Ranger policies . **Best for organizations already in the Hadoop/Hive ecosystem**—but it is not a general-purpose data warehouse RLS solution.
- **python-authz** — Framework-agnostic **ReBAC/RBAC/ABAC authorization engine** implementing Zanzibar-style relationship graphs (MIT-style) . Can express row-level access as relationship tuples (e.g., "alice can view document:roadmap because she is viewer of folder:workspace")—**not a database RLS proxy**, but a policy engine for application-level row filtering . Early release (v0.5), SQLite/PostgreSQL backends, FastAPI integration .
- **Superset Row-Level Security** — Apache Superset's native RLS feature filters rows for virtual datasets; **critical limitation**: it affects only virtual tables created by Superset, not direct SQL Lab access to underlying tables . For SQL Lab restrictions, consider database-level ACLs with separate connections or Ranger impersonation . **Not a general RLS solution**—it is a BI-layer filter.

**Frameworks for building custom systems**: For **proxy-based enforcement across heterogeneous databases**, AccessFlow provides the most complete open-source foundation . For **PostgreSQL-native declarative RLS**, rlsalchemy brings policies into SQLAlchemy/Alembic workflows . For **application-level relationship-based authorization**, python-authz implements Zanzibar-style ReBAC . For **Hadoop-ecosystem RLS**, Apache Ranger . **No open-source project matches Immuta's or Privacera's centralized policy authoring across dozens of platforms**—those remain commercial differentiators . And for warehouse-native needs, Snowflake Row Access Policies and Databricks ABAC provide built-in enforcement (platform features, not open-source products) .

## How to Contribute

1. Fork the repo.
2. Add/edit entries in `README.md` (follow existing format).
3. Include: name, link, 1–2 sentence description, and whether it's SaaS or open-source.
4. Submit PR with a short explanation.

Star the repo if you find it useful!

## Disclaimer

- This is a **community-curated** list — not exhaustive and not an endorsement.
- Row-level security platforms handling sensitive data must comply with applicable privacy regulations (GDPR, CCPA) and industry compliance requirements; **row-level policies are enforced by the database or proxy, not by client applications**—misconfiguration can expose data across tenants.
- **Row-level security carries real performance costs**—Snowflake's row access policies add 10–30% overhead on large tables unless the policy column aligns with the clustering key ; Databricks recommends simple expressions, fewer masks, and deterministic functions to preserve optimizer push-down . Test with representative workloads before production deployment.
- **Open-source row-level security is emerging but fragmented**—AccessFlow provides a proxy-based enforcement point , and rlsalchemy provides PostgreSQL-native declarative RLS . However, **no open-source project matches Immuta's or Privacera's breadth of platform connectors and centralized policy authoring**—organizations requiring multi-platform governance should evaluate commercial platforms .
- Verify license terms for commercial use; AccessFlow is open-source (check repository) , and rlsalchemy is a PyPI package (check license) .

---

**Made for data security architects, platform engineers, and organizations enforcing least-privilege access at the row level.**  
Let's make row-level data security more open, but with clear eyes about the performance overhead and governance-breadth gaps that commercial platforms still fill.
