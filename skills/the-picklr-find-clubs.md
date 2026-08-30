---
name: the-picklr-find-clubs
description: >-
  Look up The Picklr's indoor pickleball club locations from the company's own public API —
  the full national directory, a search by name or city, or a single club's page and details.
  Use this instead of scraping thepicklr.com.
api: The Picklr WordPress REST API
base_url: https://thepicklr.com/wp-json
auth: none
operations:
  - get_wp_v2_location
  - get_wp_v2_location_id
  - get_wp_v2_search
---

# Find The Picklr clubs

The Picklr's club directory is readable with no credential. As of 2026-08-30 the collection
held **157 location records**.

## 1. List the directory

`GET /wp/v2/location` (`get_wp_v2_location`)

Page through it — the collection is far larger than one page:

- `per_page` is capped at **100** and defaults to 10. Ask for 100.
- `page` starts at 1.
- Read `X-WP-Total` and `X-WP-TotalPages` from the **response headers** to know when to stop.
  Do not guess the page count, and do not loop until you get an empty array.

```
GET https://thepicklr.com/wp-json/wp/v2/location?per_page=100&page=1
```

Keep the payload small with a sparse fieldset:

```
?per_page=100&_fields=id,slug,title,link,date
```

## 2. Search for a club

`GET /wp/v2/location?search=<term>` narrows the collection by free text.

For a cross-type search use `get_wp_v2_search`:
`GET /wp/v2/search?search=<term>&subtype=location`.

Names in this dataset are club names, not city names, so a city query may miss. If a search
returns nothing, fall back to listing the full directory and matching client-side — 157
records is a small set.

## 3. Fetch one club

`GET /wp/v2/location/{id}` (`get_wp_v2_location_id`) with an `id` taken from the collection.
Never construct an id; ids are opaque integers unique to this site.

Add `?_embed` to inline the featured image rather than making a second call to
`get_wp_v2_media_id`.

## Rules

- **Read-only.** There is no write, booking or membership operation on this API. If the user
  wants to reserve a court or join, send them to the club's page — reservations run on
  PlayByPoint, a separate platform, and nothing here can make or cancel one.
- **No rate-limit headers are returned.** Back off exponentially on any non-2xx and do not
  hammer the collection; there is no `Retry-After` to read.
- **Errors are not RFC 9457.** Branch on the `code` string in
  `{"code":..., "message":..., "data":{"status":...}}`. `rest_no_route` means you used the
  wrong `rest_base` — the post type is `location`, singular, not `locations`.
- An XML mirror of the directory exists at https://thepicklr.com/location-sitemap.xml if you
  need URLs only.
