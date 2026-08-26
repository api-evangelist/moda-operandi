---
generated: '2026-08-25'
method: probed
source: https://search.modaoperandi.com/graphql
---

# Moda Operandi Search API — GraphQL

`search.modaoperandi.com/graphql` is the first-party GraphQL API that powers search, browse and
merchandising on modaoperandi.com. It is **not documented for third-party use** — Moda Operandi
publishes no developer portal, no API reference and no OpenAPI definition — but the endpoint is
public, is named by the storefront's own client configuration as `SEARCH_API_GRAPHQL_ENDPOINT`,
and answers anonymous introspection.

## How this schema was obtained

A standard `IntrospectionQuery` was POSTed to `https://search.modaoperandi.com/graphql` on
2026-08-25 with no credentials. It returned HTTP 200 and a complete `__schema`. The raw
introspection response is saved verbatim at
[`moda-operandi-search-introspection.json`](moda-operandi-search-introspection.json); the SDL in
[`moda-operandi-search-schema.graphql`](moda-operandi-search-schema.graphql) is rendered from it
without edits, additions or inference. Nothing in this directory is authored.

## Ownership

`search.modaoperandi.com` is a subdomain of the company's own registrable domain, and the endpoint
is the value of `SEARCH_API_GRAPHQL_ENDPOINT` in the runtime configuration embedded in
`https://www.modaoperandi.com/`. Every domain type in the schema is Moda Operandi's own vocabulary
— `Trunkshow`, `Designer`, `Look`, `Variant`, `NewTrunkshowPage` — with `Trunkshow` being the
company's signature product concept.

## Shape

| | |
|---|---|
| Endpoint | `https://search.modaoperandi.com/graphql` |
| Roots | `Query` only — **no mutation, no subscription** (read-only surface) |
| Query fields | 37 |
| Types | 121 (82 object, 18 enum, 10 input, 6 union, 5 scalar) |
| Deprecated fields | 47 |
| Auth | none required for introspection or query execution |
| Backing engine | Algolia (named in `dynamic_reranking`, `optional_filters`, `rule_contexts` field descriptions) |
| Server version | `{ version }` returns `2` |

## Query surface

**Search + autocomplete** — `search(input: KeywordSearchInput)`, `autocomplete(q, limit)`

**Product listing / collections** — `product_listing`, `shop`, `shop_by_variants`, `new`, `sale`,
`favorites`, `guide`, `guide_page`, `employee`, `designer_collection`, `designers_az`. All take a
`CollectionInput` and return a `CollectionResult` carrying facets plus a `VariantConnection`.

**Designers** — `designer(id|slug)`, `designers(ids, slugs, gender, vertical, all)`

**Trunkshows** — `trunkshow(id|slug)`, `trunkshows(input: TrunkshowCollectionInput)`,
`trunkshows_by_id`, `trunkshows_by_date(start_dates, end_dates)`, `trunkshowPage`,
`new_trunkshow_page`

**Looks** — `look(id)`, `looks(ids)`, `editorial_slides(ids)`

**Variants / products** — `variant(id|href|product_id)`, `variants(ids, hrefs, product_ids, query)`

**Recommendations (Algolia)** — `bought_together`, `looking_similar`, `related_products`,
`trending_items`, `trending_facets_value`

**Merchandising / navigation** — `displayPage`, `gift_categories`, `navigator_verticals`

**Operational** — `version`, `user_token`, `usage_stats(token)`

## Working example

```bash
curl -s https://search.modaoperandi.com/graphql \
  -H 'Content-Type: application/json' \
  -d '{"query":"{ version }"}'
# => {"data":{"version":2}}
```

## Caveats for anyone building on this

- This is an **undocumented internal surface**. There is no published stability, deprecation or
  rate-limit contract; 47 fields already carry `@deprecated` with no dated sunset policy behind them.
- It is **read-only**. Cart, checkout, account and order operations are not here — the storefront
  routes those to a separate REST host, `https://api.modaoperandi.com/public` (named as `MODA_API_URL`
  in the same runtime configuration), which returns 404 for every path probed anonymously and
  publishes no specification.
- Use is governed by https://www.modaoperandi.com/terms, not by any API-specific agreement.
