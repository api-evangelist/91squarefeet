---
name: 91squarefeet-read-company-editorial
description: >-
  Retrieve 91Squarefeet's published editorial — blog posts, case studies, press releases and testimonials —
  from the public read-only content API on 91squarefeet.com, with authors, categories and excerpts resolved.
api: 91squarefeet:91squarefeet-content-api
base_url: https://91squarefeet.com/wp-json
auth: none
operations:
  - listPosts
  - getPost
  - listCaseStudy
  - getCaseStudy
  - listPressRelease
  - getPressRelease
  - listTestimonial
  - listCategories
  - listUsers
  - search
generated: '2026-09-05'
method: generated
source: openapi/91squarefeet-content-api-openapi.yml
---

# Read 91Squarefeet's published editorial

Four separate collections carry 91Squarefeet's public writing. They are different content types, not tags on
one type, so a complete answer usually needs more than one call.

| Collection | Operation | Records observed 2026-09-05 | What it holds |
|---|---|---|---|
| `posts` | `listPosts` | 39 | Blog posts on retail expansion, office design, fit-out practice |
| `case_study` | `listCaseStudy` | 3 | Client case studies |
| `press_release` | `listPressRelease` | 16 | Press releases and media mentions |
| `testimonial` | `listTestimonial` | 8 | Client testimonials |

## Steps

### 1. Search across everything first — `search`

```
GET https://91squarefeet.com/wp-json/wp/v2/search?search={terms}&per_page=100
```

Returns `{id, title, url, type, subtype}` per hit. `subtype` tells you which collection the record lives in,
so use it to route the follow-up call. This is the cheapest way to find out whether the company has written
about a topic at all.

### 2. Walk a specific collection

```
GET https://91squarefeet.com/wp-json/wp/v2/posts?per_page=100&_embed&orderby=date&order=desc
```

- Page with `page` / `per_page` (max 100). `X-WP-Total` and `X-WP-TotalPages` come back as headers, and
  `Link: rel="next"` is authoritative — follow it rather than computing the next page.
- Narrow by date with `after` / `before` (ISO 8601), by term with `categories`, or free-text with `search`.
- Trim the payload with `_fields=id,title,link,date,excerpt` when you only need a listing.

Posts carry `content.rendered` and `excerpt.rendered` as **HTML strings** — strip or render them, do not
treat them as plain text. `case_study` and `press_release` records return `excerpt` but not `content`, so
follow `link` to the page when you need the full body.

### 3. Resolve authors and categories

```
GET https://91squarefeet.com/wp-json/wp/v2/users
GET https://91squarefeet.com/wp-json/wp/v2/categories?per_page=100
```

`posts.author` is a user id; `posts.categories`, `case_study.categories` and `press_release.categories` are
arrays of category term ids. Fetch both maps once per session and join locally — or just pass `_embed` and
read `_embedded['wp:term']` and `_embedded.author`.

## Notes

- Anonymous, read-only, no key. Send no `Authorization` header.
- Responses are served `Cache-Control: no-store` with no `ETag` or `Last-Modified`, so conditional requests
  are not available — cache on your own side if you poll.
- 91Squarefeet also publishes an `llms.txt` at `https://91squarefeet.com/llms.txt` (Rank Math generated) that
  lists every post and page with a one-paragraph summary. For a quick inventory that is cheaper than paging
  this API; for structured fields, use the API.
- Errors use the WordPress envelope `{code, message, data.status}` — see
  `errors/91squarefeet-problem-types.yml`.
