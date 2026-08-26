# Moda Operandi

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Moda Operandi is a New York-based luxury fashion e-commerce marketplace founded in 2010 by Lauren
Santo Domingo and Aslaug Magnusdottir. It is built around the *trunkshow* model — customers
pre-order directly from designers' full runway collections weeks before the clothes reach stores —
alongside an in-season boutique carrying more than 1,000 brands across ready-to-wear, fine
jewelry, home and beauty.

## What this profile found

**Moda Operandi runs no developer program.** There is no developer portal, no API documentation,
no API reference, no OpenAPI or Swagger definition, no SDK, no CLI, no webhook or event surface,
no MCP server, no A2A agent card, no `/.well-known/` document on any host, no status page, no
changelog, no `security.txt`, no published rate limits and no API pricing.

**But one real machine-readable contract exists.** `https://search.modaoperandi.com/graphql` is the
first-party GraphQL API that powers search and browse on modaoperandi.com. It is undocumented, but
it is public, it is named by the storefront's own runtime configuration as
`SEARCH_API_GRAPHQL_ENDPOINT`, and it answers anonymous introspection. The complete schema —
37 query fields, 121 types, read-only (no mutation or subscription root) — was introspected on
2026-08-25 and is saved verbatim in [`graphql/`](graphql/).

Notable findings recorded in the artifacts:

- **Twelve of the 37 root query fields are already `@deprecated`** with no dates and no removal
  policy; ten of them say "use `product_listing` instead". See [`lifecycle/`](lifecycle/).
- **Error responses leak a full server-side stacktrace** with absolute filesystem paths under
  `/opt/app/dist/server/`. See [`errors/`](errors/).
- **No rate-limit headers of any kind** are emitted, so a client gets no back-off signal. See
  [`rate-limits/`](rate-limits/).
- **No API SDK exists.** The nine first-party npm packages under the `@moda` and `@mo-tools-engine`
  scopes are a React design system, design tokens and an abandoned internal-tools devkit — none of
  them wraps any Moda Operandi API. See [`packages/`](packages/).

## Links

- Website: https://www.modaoperandi.com/
- Terms: https://www.modaoperandi.com/terms
- Privacy: https://www.modaoperandi.com/privacy
- GitHub: https://github.com/ModaOperandi
- Help Center: https://help.modaoperandi.com/hc/en-us
