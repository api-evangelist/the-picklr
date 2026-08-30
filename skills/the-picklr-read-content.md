---
name: the-picklr-read-content
description: >-
  Read The Picklr's published content — blog posts, site pages, press coverage and events —
  from the company's public API, with correct pagination, filtering and field selection.
api: The Picklr WordPress REST API
base_url: https://thepicklr.com/wp-json
auth: none
operations:
  - get_wp_v2_posts
  - get_wp_v2_posts_id
  - get_wp_v2_pages
  - get_wp_v2_pages_id
  - get_wp_v2_press
  - get_wp_v2_event
  - get_wp_v2_categories
  - get_wp_v2_tags
  - get_wp_v2_types
---

# Read The Picklr content

All collections below are anonymously readable. Observed counts on 2026-08-30: 23 posts,
53 pages, 5 press items, 4 events.

## Choose the collection

Call `get_wp_v2_types` first if you are unsure what exists — it returns the live registry of
post types and, critically, each type's `rest_base`. The `rest_base` is the path segment, and
it is **not** always the plural of the label: Locations is served at `/wp/v2/location`.

| Content | Operation | Path |
|---|---|---|
| Blog posts | `get_wp_v2_posts` | `/wp/v2/posts` |
| Site pages | `get_wp_v2_pages` | `/wp/v2/pages` |
| Press coverage | `get_wp_v2_press` | `/wp/v2/press` |
| Events | `get_wp_v2_event` | `/wp/v2/event` |

## Pagination

Page-number only; there is no cursor.

- `per_page` max 100, default 10
- `page` from 1
- `X-WP-Total` / `X-WP-TotalPages` response headers give the real size
- A `Link` header carries `rel="next"` — follow it rather than incrementing blindly

## Narrowing

- `search=<term>` — free text
- `after` / `before` / `modified_after` / `modified_before` — ISO 8601 date-times
- `categories` / `tags` — arrays of term ids from `get_wp_v2_categories` and `get_wp_v2_tags`
- `orderby` with `order=asc|desc`
- `_fields=id,title,link,date` — return only what you need
- `_embed` — inline author, featured media and terms in one round trip

## Rules

- **`context=edit` will fail.** It returns `401 rest_forbidden` for anonymous callers. Stay on
  the default `context=view`.
- Titles and content come back as `{"rendered": "<html>"}`. Strip the HTML before quoting to a
  user; do not present raw markup.
- This is a marketing site's content API, not a product changelog. Do not present blog posts as
  release notes.
- Do not attempt writes. `POST`/`PUT`/`DELETE` require a WordPress Application Password issued
  from the site admin and are not available to third parties.
