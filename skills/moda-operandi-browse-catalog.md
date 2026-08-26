---
name: Browse the Moda Operandi catalog with facets and pagination
description: >-
  Query the Moda Operandi GraphQL search API for a faceted, paginated product listing — the same
  call the modaoperandi.com storefront makes to render a category page — and read the facet
  buckets back to build filters.
api: graphql/moda-operandi-search-schema.graphql
endpoint: https://search.modaoperandi.com/graphql
operations:
  - Query.product_listing
  - Query.variant
  - Query.designers
method: generated
source: graphql/moda-operandi-search-schema.graphql
verified: every query below was executed live against the endpoint on 2026-08-25 and returned data
---

# Browse the Moda Operandi catalog

## Before you start

- The endpoint is `https://search.modaoperandi.com/graphql`. POST JSON. No credentials are needed.
- This API is **undocumented and internal**. Moda Operandi publishes no developer program, no
  terms covering programmatic use, and no rate limits. Be conservative: serialize your requests,
  do not crawl the whole catalog, and stop on any error rather than retrying in a loop.
- It is **read-only**. There is no mutation root. You cannot add to a cart, place an order, or
  change anything.

## Step 1 — ask for a listing

Use `product_listing`. Do **not** use `shop`, `new`, `sale`, `search`, `favorites`, `guide`,
`guide_page`, `employee`, `shop_by_variants` or `designer_collection` — all nine are `@deprecated`
in the schema with the reason "use product_listing instead".

Pick the section with `section_enum`, one of: `DESIGNER`, `EMPLOYEE`, `FAVORITES`, `GIFTS`,
`GUIDE`, `NEW`, `SALE`, `SEARCH`, `SHOP`, `SHOP_BY_VARIANTS`.

```graphql
query Listing($input: CollectionInput) {
  product_listing(section_enum: SHOP, client_country_code: "US", input: $input) {
    page_name
    total_items
    designers { name slug total_items }
    variants(page: 1, per_page: 24, sort_enum: RECENCY) {
      pagination { page per_page total_entries total_pages }
      variants { id designer_name designer_slug description availability_enum }
    }
  }
}
```

```json
{ "input": { "category_path": "Clothing", "country_code": "US", "on_sale": false } }
```

**The two country arguments are not the same field and this is the easiest mistake to make.**
`client_country_code` is an argument on the *root field*. `country_code` is a field on the
*`CollectionInput` object*. Putting `client_country_code` inside `input` returns
`BAD_USER_INPUT: Field "client_country_code" is not defined by type "CollectionInput"`.

## Step 2 — paginate

Pagination arguments live on the **connection** (`variants`), never on the root query. Use the
page-number form:

- `page`, `per_page` → read back `pagination { page per_page total_entries total_pages }`
- Do **not** use `after` / `pageInfo { endCursor hasNextPage }`. Those Relay fields exist but are
  `@deprecated("use pagination instead")`.
- `sort_enum` accepts `DEFAULT`, `HIGH`, `INVENTORY`, `LOW`, `MARKDOWN`, `RECENCY`, `RECENCY_NEW`.
  A legacy string `sort` argument still exists; prefer the enum.

Walk pages until `page == total_pages`. Do not request `all: true` on a large category — the
`SHOP` / `Clothing` listing alone reported `total_items: 22280`.

## Step 3 — read the facets

`CollectionResult` returns every filter bucket the storefront renders, in the same response:
`designers`, `designers_with_id`, `attributes`, `primary_attributes`, `sizes`, `materials`,
`filter_colors`, `collections`, `sale_tags`, `seasons`, `subcategories`, `tags`, `category_tree`,
`availability`, `rounded_markdown_percentages`, plus `applied_filters`. Each bucket carries
`total_items`, so you can build a complete filter UI from one round trip.

Facet types differ — `designers` and `attributes` are `NameSlugFacet` (`name`, `slug`,
`total_items`), plain `collections`/`tags` are `Facet` (`name`, `total_items`), and `sizes` is
`SizeFacet`. Selecting `name` on `SizeFacet` fails validation. Check the SDL before you select.

## Step 4 — narrow

Feed slugs from step 3 straight back into `CollectionInput`: `designers: [...]`,
`attributes: [...]`, `primary_attributes: [...]`, `sizes: [...]`, `materials: [...]`,
`filter_colors: [...]`, `collections: [...]`, `category_path`, `price_range: { min, max }`,
`on_sale` / `no_sale` / `sale`, `availability_enum: [AVAILABLE, AVAILABLE_NOW, PREORDER, SOLD_OUT,
TRUNKSHOW]`.

Leave `rules`, `rule_contexts`, `optional_filters` and `dynamic_reranking` alone. Their own schema
descriptions say they bypass Moda Operandi's merchandising rules and preview unpublished changes;
they are console affordances, not consumer parameters.

## Step 5 — fetch one product

```graphql
{ variant(id: "729360") { id designer_name designer_slug availability_enum vertical } }
```

`variant` also accepts `href` or `product_id` instead of `id`. Prefer `availability_enum` over
`availability`, `vertical` over `gender`, and `master_variants_data` over `current_variants_data` —
the older field in each pair is deprecated.

## Errors

Validation failures come back as HTTP 200 with a GraphQL `errors[]` array and
`extensions.code` of `GRAPHQL_VALIDATION_FAILED` or `BAD_USER_INPUT`. An empty or missing query is
HTTP 400 `BAD_REQUEST`. The error body includes a server stacktrace — ignore it; do not log it.
See `errors/moda-operandi-problem-types.yml`.
