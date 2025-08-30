_Last updated: 2025-08-28 • [Edit this page](https://github.com/Vitae-Springs-Software-LLC/

APIExtender/edit/main/docs/bc-api-v2-vs-extended-connector.md)_

# Business Central API v2.0 vs Extended Connector — Practical Q&A

_Audience:_ Controllers, accountants, FP&A, owners/C‑level at SMBs, and business analysts. The goal is to clarify **what the out‑of‑the‑box (OOB) Business Central APIs provide** versus **what an extended connector exposes**, especially for **Power BI** users who need reliable access to the right tables/fields (including extension data and dimensions).

---

## TL;DR (Plain language)
- The **standard Business Central API v2.0** exposes core entities (customers, vendors, items, documents, etc.). **Dimensions are not simple columns**; they live in related **`dimensionSetLines`** that you must expand/join. **Extension tables and custom fields are not included.**
- An **extended connector** publishes **your extension tables and fields as first‑class endpoints** and can provide **dimension data in a more analysis‑friendly shape** (e.g., pre‑flattened columns or ready‑to‑merge views), which is much easier to use in Power BI.

---

## Q1. What do I get out of the box with Business Central API v2.0?
**Short answer:** Standard REST/OData endpoints for core tables only. You can read/write those entities, but **dimensions** are exposed via a child collection (`dimensionSetLines`), and **extension data isn’t included**.

**Key characteristics:**
- **URL shape:** `/v2.0/{tenant}/{environment}/api/v2.0/companies({companyId})/...`
- **Entities:** customers, vendors, items, documents (sales orders/invoices, purchase orders/invoices), ledger entries, journals, etc.
- **Dimensions:** attached through `dimensionSetLines` — not scalar columns on the header/line.
- **Filtering/paging:** Standard OData (`$filter`, `$select`, `$orderby`, `$top`), `@odata.nextLink` for paging.
- **Security:** Azure AD (OAuth). Per‑tenant and per‑environment.
- **Limitations for reporting:**
  - **No extension tables or custom fields.**
  - Dimensions require **indirect access** (expand, navigate, and sometimes multiple requests), which complicates Power BI models.

---

## Q2. Why do people hit a wall in Power BI with the OOB API?
Because many reports assume **flat fields**. In BC v2.0, **dimensions are separate rows** in `dimensionSetLines` and some records can only be reached **indirectly**. That means:
- You must learn OData navigation to pull dimensions.
- You’ll often need multiple queries or merges to shape data.
- Standard endpoints **won’t carry your extension fields** used by customizations (e.g., Shopify, ISV apps, or internal tables).

This is manageable for developers, but it’s **friction** for analysts/finance teams who just want reliable columns to slice by.

---

## Q3. What does an extended connector change?
An extended connector (via custom API pages/queries) typically:
- **Publishes extension tables and fields** as their own endpoints.
- Provides **dimension data in a more analysis‑friendly shape**, e.g.:
  - pre‑flattened columns like `GlobalDimension1Code`, `GlobalDimension2Code`, or
  - curated views that join `dimensionSetLines` for you.
- Preserves OData semantics so Power BI can consume it **with fewer merges** and less modeling effort.

Net effect: fewer moving parts to get to a usable model; fewer custom steps; more predictable schemas across tenants.

---

## Q4. Concrete examples (side‑by‑side)

### Example A — Read a sales order **with dimensions** (OOB)
**What the standard v2.0 API requires:** use `$expand` to fetch dimensions.

```http
GET https://api.businesscentral.dynamics.com/v2.0/{tenant}/{environment}/api/v2.0/companies({companyId})/salesOrders
  ?$select=id,number,externalDocumentNumber
  &$expand=dimensionSetLines($select=code,valueCode,valueDisplayName)
```
**Notes:**
- Dimension values are in a **child array** per document.
- Filtering by a dimension uses `any()`:
  ```http
  .../salesOrders?$filter=dimensionSetLines/any(d:d/code eq 'DEPARTMENT' and d/valueCode eq 'SALES')
  ```
- In Power BI, you’ll likely **expand the child table** and then **transform** rows → columns.

### Example B — The same outcome via an **extended connector**
**What an extended endpoint can expose:** a document view with **dimension columns already joined**.

```http
GET https://{your-endpoint}/api/ext/v1.0/companies({companyId})/SalesOrdersWithDims
  ?$select=id,number,externalDocumentNumber,GlobalDimension1Code,GlobalDimension2Code
```
**Notes:**
- No `$expand` or `any()` needed for common slicing.
- Analysts can filter/summarize by `GlobalDimension1Code` directly.
- You can still include a related `dimensionSetLines` navigation if you need **all** dimensions beyond the globals.

### Example C — **Extension field** on Items
**OOB:** The standard `items` endpoint won’t include your custom field (e.g., `ShelfLifeDays`).

```http
// Standard OOB — field not present
GET .../api/v2.0/companies({companyId})/items?$select=id,number,displayName
```

**Extended:** A custom Items API that exposes the field as a first‑class column.

```http
// Extended connector — field available for analytics
GET https://{your-endpoint}/api/ext/v1.0/companies({companyId})/API_ItemExt
  ?$select=id,number,displayName,ShelfLifeDays,ItemCategoryExt
```
**Notes:**
- No extra merges to retrieve custom data.
- Power BI schema remains stable even across tenants that share the extension.

---

## Q5. What does this mean for report authors and data teams?
- **Faster time‑to‑value:** Fewer M‑code merges and transforms; fewer OData navigation quirks.
- **Consistency:** A schema that doesn’t break when extension fields are required.
- **Lower skill barrier:** Finance/ops users can self‑serve more effectively; developers spend less time “plumbing” and more time on actual metrics.

---

## Q6. Suggested Power BI patterns (minimal friction)

### Pattern 1 — OOB: Expand and pivot dimensions
Use one query for documents and an **expanded** child table for `dimensionSetLines`, then pivot key dimension codes into columns. Keep a small map of the codes you care about (e.g., Department, CustomerGroup).

### Pattern 2 — Extended: Direct columns + optional detail
Point visuals to the **flattened** columns (`GlobalDimension1Code`, etc.) and keep a secondary table for full `dimensionSetLines` when auditors need the raw detail.

### Pattern 3 — Extension fields
Load from the **extended endpoints** that include your custom fields. Avoid mixing OOB endpoints with extension endpoints for the **same entity** in the same model unless you control the joins carefully.

---

## Q7. When should I stick to OOB vs use an extended connector?
- **OOB is fine** when you only need core entities and a couple of dimensions that you’re comfortable expanding and shaping in Power BI.
- **Extended connector** is warranted when any of the following are true:
  - Reports must include **extension tables/fields**.
  - Analysts shouldn’t have to write OData navigation or complex M.
  - You want **repeatable, tenant‑agnostic** models that avoid per‑tenant tweaking.

---

## Appendix — Two ready‑to‑use snippets

### A1) OOB Power Query (M) — Sales orders with dimensions (flatten common codes)
```m
let
  Source = OData.Feed(
    "https://api.businesscentral.dynamics.com/v2.0/" & Tenant & "/" & Environment & "/api/v2.0/",
    null, [Implementation="2.0"]),
  Company = Source{[Name="companies"]}[Data],
  CompanyFiltered = Table.SelectRows(Company, each [id] = CompanyId),
  SalesOrders = CompanyFiltered{0}[Data]{[Name="salesOrders"]}[Data],
  KeepCols = Table.SelectColumns(SalesOrders,{"id","number","externalDocumentNumber","dimensionSetLines"}),
  ExpandDims = Table.ExpandTableColumn(
      KeepCols,
      "dimensionSetLines",
      {"code","valueCode"},
      {"DimCode","DimValue"}
  ),
  // Keep only codes you want to flatten (e.g., Department, Project)
  FilterWanted = Table.SelectRows(ExpandDims, each List.Contains({"DEPARTMENT","PROJECT"}, [DimCode])),
  Pivot = Table.Pivot(FilterWanted, List.Distinct(FilterWanted[DimCode]), "DimCode", "DimValue"),
  Result = Table.RemoveColumns(Pivot, {"dimensionSetLines"})
in
  Result
```

### A2) Extended connector — Simple load
```m
let
  Source = OData.Feed(
    "https://{your-endpoint}/api/ext/v1.0/",
    null, [Implementation="2.0"]),
  Company = Source{[Name="companies"]}[Data],
  CompanyFiltered = Table.SelectRows(Company, each [id] = CompanyId),
  SalesOrders = CompanyFiltered{0}[Data]{[Name="SalesOrdersWithDims"]}[Data],
  Result = Table.SelectColumns(SalesOrders,{
    "id","number","externalDocumentNumber",
    "GlobalDimension1Code","GlobalDimension2Code"
  })
in
  Result
```

---

### Final note (no spin)
If you’re comfortable with OData and Power Query shaping, you can absolutely build reliable models on the **OOB v2.0 APIs**. If your reporting depends on **extension data** and you want clean, repeatable models with **less M‑code plumbing**, use an **extended connector** that exposes those fields and (optionally) pre‑flattens commonly used dimensions.

