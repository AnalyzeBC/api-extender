🚀 Getting Started (quick)

Install the .app in Business Central: Extensions → Manage → Upload.

Activate using the provided activation code (Business Central posts to your APIM /apibuilder/activate which forwards to Functions /api/activate).

Use the APIs (standard OData v4) or the extended endpoints for your custom tables/fields and simplified dimensions.

🧩 Files in this repo

APIBuilder.app — the extension package

defaults.csv — example/default configuration

docs/bc-api-v2-vs-extended-connector.md — the Q&A explainer for analysts/finance/owners

🧠 Notes for Power BI authors

OOB BC API v2.0 exposes dimensions via dimensionSetLines (child rows).

The extended connector can provide pre-flattened dimension columns and expose extension fields directly, reducing M-code merges and making models tenant-agnostic.

🛠️ Support

Issues and clarifications: open a GitHub Issue on this repo.

Feel free to PR doc improvements (prefer squash merge for a clean history).