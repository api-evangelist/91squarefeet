---
name: 91squarefeet-browse-project-portfolio
description: >-
  Retrieve 91Squarefeet's delivered fit-out projects, the brands they were delivered for, and the imagery
  attached to each, from the public read-only content API on 91squarefeet.com. Use this to answer questions
  about what 91Squarefeet has built, for whom, and where.
api: 91squarefeet:91squarefeet-content-api
base_url: https://91squarefeet.com/wp-json
auth: none
operations:
  - listPortfolio
  - getPortfolio
  - listClient
  - getClient
  - listClientCategory
  - getMediaItem
generated: '2026-09-05'
method: generated
source: openapi/91squarefeet-content-api-openapi.yml
---

# Browse the 91Squarefeet project portfolio

91Squarefeet is a turnkey retail and office fit-out contractor in India. It publishes no developer program,
but its public website serves its portfolio, client list and project imagery over an unauthenticated
read-only content API. Everything below uses operations that exist in
`openapi/91squarefeet-content-api-openapi.yml`.

## Before you start

- **No credentials.** Send no `Authorization` header. Reads succeed anonymously; the origin answers
  `Allow: GET` on these collections.
- **This is read-only.** There is no write, no reversal and no idempotency concern — nothing you do here
  changes state.
- **Be polite.** No rate limit is published and none is signalled. Keep concurrency low and page rather than
  hammering.

## Steps

### 1. List delivered projects — `listPortfolio`

```
GET https://91squarefeet.com/wp-json/wp/v2/portfolio?per_page=100&_embed
```

- `per_page` maximum is **100**. Asking for more returns `400 rest_invalid_param` with
  `data.details.per_page.code = rest_out_of_bounds`.
- Read `X-WP-Total` and `X-WP-TotalPages` from the response headers to size the walk, then increment `page`.
  Follow `Link: <...>; rel="next"` instead of guessing.
- `_embed` inlines the featured image and taxonomy terms under `_embedded`, which saves a second round trip.
- Useful filters that the contract declares: `search`, `orderby` + `order`, `slug`, `include`, `exclude`,
  `after` / `before` (ISO 8601), `client_category`.
- 42 project records were present when this skill was written.

Each record carries `id`, `title.rendered`, `slug`, `link` (its page on 91squarefeet.com), `date`,
`featured_media`, and the term id arrays `client_category`, `display_location`, `mobile_category`.

### 2. Retrieve one project — `getPortfolio`

```
GET https://91squarefeet.com/wp-json/wp/v2/portfolio/{id}
```

`{id}` is the bare integer from step 1. Ids are site-scoped with no type prefix, so an id is only meaningful
together with its collection — never carry an id from one collection to another.

### 3. Resolve the brand — `listClient` / `getClient`

```
GET https://91squarefeet.com/wp-json/wp/v2/client?per_page=100
```

73 client records were present when this skill was written. Portfolio and Client records share the
`client_category`, `display_location` and `mobile_category` taxonomies, so match them on term ids rather than
on name strings.

### 4. Resolve category names — `listClientCategory`

```
GET https://91squarefeet.com/wp-json/wp/v2/client_category?per_page=100
```

Returns `{id, name, slug, count, taxonomy}` per term. Build the id→name map once and reuse it.

### 5. Resolve imagery — `getMediaItem`

```
GET https://91squarefeet.com/wp-json/wp/v2/media/{featured_media}
```

Use `source_url` for the original file and `media_details.sizes` for generated renditions. Skip this step
entirely if you passed `_embed` in step 1.

## Handling errors

The API returns the WordPress envelope, **not** RFC 9457 problem+json:

```json
{"code":"rest_invalid_param","message":"Invalid parameter(s): per_page","data":{"status":400,"params":{...}}}
```

| code | status | what to do |
|---|---|---|
| `rest_invalid_param` | 400 | Read `data.params` for the failing parameter and its constraint, then retry with a valid value. |
| `rest_forbidden` | 401 | You reached an authenticated namespace. Nothing in this skill needs auth — check the path. |
| `rest_no_route` | 404 | The path or method does not exist. Check it against `https://91squarefeet.com/wp-json`. |

## What this API is not

It is the company's **website content**. It carries no project schedules, costs, supplier records, quotes or
customer data, and it is not a construction or project-management API. If you need those, 91Squarefeet
publishes no interface for them — the only commercial path is the contact form at
`https://91squarefeet.com/contact-us/`.
