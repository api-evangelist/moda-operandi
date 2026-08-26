---
name: Work with Moda Operandi trunkshows and looks
description: >-
  Read Moda Operandi's signature pre-order surface — designer trunkshows, the runway looks inside
  them, and the variants each look contains — from the GraphQL search API.
api: graphql/moda-operandi-search-schema.graphql
endpoint: https://search.modaoperandi.com/graphql
operations:
  - Query.trunkshows
  - Query.trunkshow
  - Query.trunkshows_by_date
  - Query.trunkshows_by_id
  - Query.look
  - Query.looks
method: generated
source: graphql/moda-operandi-search-schema.graphql
verified: every query below was executed live against the endpoint on 2026-08-25 and returned data
---

# Trunkshows and looks

A **trunkshow** is a designer's full runway collection offered for pre-order weeks before it
reaches stores — the concept Moda Operandi was founded on. Inside a trunkshow are **looks** (one
styled runway outfit each), and inside a look are **variants** (buyable SKUs).

## Step 1 — list the active trunkshows

```graphql
{
  trunkshows(input: { gender: women }) {
    total_items
    designers { name slug total_items }
    categories { name slug total_items }
    trunkshows(page: 1, per_page: 12) {
      pagination { page per_page total_entries total_pages }
      trunkshows { id name slug season season_name look_count restricted }
    }
  }
}
```

`TrunkshowCollectionInput` accepts `gender` (a `Gender` enum whose only value is `women`),
`designer_slugs`, `designer_ids`, `category`, `excluded_categories` and `single_designer_only`.
Pagination and `sort_enum: TrunkshowCollectionSort` live on the inner `trunkshows` connection, same
rule as everywhere else in this schema.

## Step 2 — fetch one trunkshow and walk its looks

```graphql
{
  trunkshow(slug: "malo-fw26") {
    id name slug season_name look_count restricted private_until_i
    active_start_i active_end_i
    designer_names designer_slugs
    primary_image_urls { __typename }
    looks(page: 1, per_page: 10) {
      pagination { total_entries total_pages }
      looks { id name position primary_image_urls { __typename } variant_ids }
    }
  }
}
```

`trunkshow` takes `id` **or** `slug`. Two timing fields matter and neither is documented anywhere
public, so read them carefully:

- `active_start_i` / `active_end_i` — unix timestamps bounding when the trunkshow is orderable.
- `private_until_i` — a unix timestamp before which the trunkshow is private.
- `restricted: Boolean!` — the trunkshow is gated to particular customer groups
  (`active_for: TrunkshowActiveFor`).

Do not surface a trunkshow to a user without checking `restricted` and the active window. A
trunkshow the API will return is not necessarily one a given shopper is entitled to see.

## Step 3 — by date, or by known ids

```graphql
{
  trunkshows_by_date(start_dates: { from: 1750000000, to: 1790000000 }, limit: 10) {
    id name slug season_name
  }
  trunkshows_by_id(slugs: ["malo-fw26"]) { id name look_count }
}
```

`start_dates` and `end_dates` take a `FromToTimestamp` of unix seconds.

## Step 4 — a single look

```graphql
{
  look(id: "387790") {
    id name title position
    trunkshow { id name slug }
    variants(page: 1, per_page: 20) { variants { id designer_name availability_enum } }
    next_look_id previous_look_id
  }
}
```

`next_look_id` / `previous_look_id` let you page through a runway sequence without re-querying the
trunkshow. Avoid `runway_image_urls` (deprecated, use `primary_image_urls`), `look_type`
(use `look_type_enum`), `trunkshow_expired` (check `trunkshow` instead), and `type`
(use `__typename`).

## Availability semantics

`Variant.availability_enum` is the field to branch on: `AVAILABLE`, `AVAILABLE_NOW`, `PREORDER`,
`SOLD_OUT`, `TRUNKSHOW`. `TRUNKSHOW` and `PREORDER` mean the item is not in stock — it is being
made after the order. The older `availability: String!` and `is_boutique: Boolean!` fields are both
deprecated in favour of it.

## Reminder

There is no mutation root. You can read a trunkshow and its looks; you cannot reserve, pre-order or
purchase anything through this API.
