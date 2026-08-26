---
name: Search and autocomplete against Moda Operandi
description: >-
  Drive the Moda Operandi type-ahead and keyword search, then pivot into Algolia-backed
  recommendations for a chosen product.
api: graphql/moda-operandi-search-schema.graphql
endpoint: https://search.modaoperandi.com/graphql
operations:
  - Query.autocomplete
  - Query.product_listing
  - Query.related_products
  - Query.looking_similar
  - Query.bought_together
  - Query.trending_items
method: generated
source: graphql/moda-operandi-search-schema.graphql
verified: every query below was executed live against the endpoint on 2026-08-25 and returned data
---

# Search and discover on Moda Operandi

## Step 1 — type-ahead

```graphql
{
  autocomplete(q: "gown", limit: 5) {
    designer_name_slugs { name slug }
    category_pages { __typename }
    trunkshow_autocomplete_facets { __typename }
  }
}
```

Returns designer suggestions with slugs, category-page suggestions, and trunkshow facets. Do not
select `query_reformulations` or `trending_queries` — both are `@deprecated("this isn't supported
anymore")`. Do not select `type`; use `__typename`.

## Step 2 — full keyword search

The root `search(input: KeywordSearchInput)` field is `@deprecated("use product_listing
instead")`. Run keyword search through `product_listing` with `section_enum: SEARCH` and put the
term in `input.q`:

```graphql
query Search($input: CollectionInput) {
  product_listing(section_enum: SEARCH, client_country_code: "US", input: $input) {
    total_items
    designers { name slug total_items }
    variants(page: 1, per_page: 24) {
      pagination { total_entries total_pages }
      variants { id designer_name designer_slug availability_enum }
    }
  }
}
```

```json
{ "input": { "q": "silk gown", "country_code": "US" } }
```

`primary_attributes` and `attributes` are ANDed with each other while values inside each bucket are
ORed — the schema states this explicitly. Use `primary_attributes` for the filter the shopper
committed to and `attributes` for secondary refinements.

## Step 3 — recommendations for a product

All four recommendation fields key off a **variant id** and are Algolia-backed:

```graphql
{
  related_products(variant_id: "729360", max_recommendations: 4) { id designer_name }
  looking_similar(variant_id: "729360", max_recommendations: 4) { id designer_name }
  bought_together(variant_id: "729360", max_recommendations: 4) { id designer_name }
}
```

Each also accepts `threshold: Float` (Algolia's confidence cutoff) and `client_country_code`. They
return `[Variant]!` — possibly empty. An empty array is a normal answer, not an error.

## Step 4 — merchandising-wide trends

```graphql
{
  trending_items(facet_name: "designers", facet_value: "proenza-schouler", max_recommendations: 5) { id designer_name }
  trending_facets_value(facet_name: "designers", max_recommendations: 10) { __typename }
}
```

## Notes

- `variants(query: "...")` exists as a root field but is a lookup by id/href/product_id with an
  optional query filter, not a search entry point; it returned `[]` for a plain keyword. Use
  `product_listing` for search.
- No credentials, no rate-limit headers, no back-off signal. Space your calls out.
