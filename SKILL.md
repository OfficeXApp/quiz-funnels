---
name: quiz-funnels
description: |
  Build and manage interactive quizzes, assessments, and lead-capture funnels with your AI agent. Create scored quizzes from JSON schemas, publish them instantly, run A/B tests, and track analytics — all through conversation.
  Use when: (1) Creating or updating a quiz/assessment/funnel, (2) Checking analytics like visitors, scores, and drop-off rates, (3) Running A/B tests on quiz variants, (4) AI-routing visitors to the right quiz variant with natural language hints, (5) Managing API keys for team access, (6) Uploading videos for quizzes, (7) Viewing individual visitor journeys and quiz scores, (8) Reviewing answer distributions, (9) Creating sandboxes to safely edit quizzes without affecting production, (10) Using the element inspector to get exact component references for AI agents.
  Triggers: quiz funnel, quiz builder, quiz scoring, quiz api, lead quiz, personality quiz, assessment quiz, funnel builder, conversion quiz, interactive quiz, split test, ab test, create quiz, ai routing, variant routing, hint routing, sandbox, element inspector, devtools
---

# Quiz Funnels

Build and manage interactive quizzes, assessments, and lead-capture funnels — directly through your AI agent. Create scored quizzes with 56+ component types, automatic grading, conditional routing based on scores, and full analytics on how visitors answer.

> **Install on OfficeX:** [officex.app/store/en/app/quiz-funnels](https://officex.app/store/en/app/quiz-funnels)

## What You Can Do

- **Create quizzes** — build scored assessments, personality quizzes, lead qualification funnels from a JSON schema
- **Publish instantly** — quizzes go live at `https://yourname.catalogs.cloud.zoomgtm.com/your-quiz-slug`
- **Check analytics** — see visitors, completion rates, score distributions, page drop-off, and revenue
- **View answer breakdowns** — see how visitors answered each question (e.g. "56% chose Option A")
- **Run A/B tests** — split traffic between quiz variants to optimize conversions
- **AI variant routing** — auto-route visitors to the best quiz variant using natural language hints
- **Sandbox editing** — clone a quiz to safely make changes without affecting the live version, then promote when ready
- **Element inspector** — hold Shift+Alt to hover-inspect any element and copy its exact `pageId/componentId` reference for AI agents
- **View visitor journeys** — trace exactly what each visitor did, including their quiz score
- **Manage access** — create API keys for team members or integrations
- **Upload videos** — add video content with automatic HLS transcoding

## Getting Started

After installing Quiz Funnels on OfficeX, you receive credentials automatically. You can also sign up at the dashboard and create API keys from Settings.

```bash
# Your API key (created from Settings page or received on install)
CF_API_KEY="cfk_..."

# Production API
CF_API_URL="https://catalog-funnel-api.cloud.zoomgtm.com"
```

### Authentication

Pass your API key as a Bearer token on all requests:

```bash
curl -H "Authorization: Bearer cfk_..." \
  https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

If you installed via OfficeX, you can also use your install credentials:

```bash
TOKEN=$(echo -n "${OFFICEX_INSTALL_ID}:${OFFICEX_INSTALL_SECRET}" | base64)
curl -H "Authorization: Bearer $TOKEN" \
  https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

---

## Managing Quizzes

Quizzes are catalogs with quiz-scored components. All the same CRUD operations apply.

### List your quizzes

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

**Response:**
```json
{
  "ok": true,
  "data": [
    {
      "catalog_id": "01HXY...",
      "slug": "marketing-quiz",
      "name": "Marketing Knowledge Quiz",
      "status": "published",
      "visibility": "public",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

### Create a quiz

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

```json
{
  "slug": "marketing-quiz",
  "name": "Marketing Knowledge Quiz",
  "schema": { ... },
  "status": "published",
  "visibility": "public"
}
```

- `slug` — URL-friendly name (lowercase, hyphens). Your quiz will be live at `https://yourname.catalogs.cloud.zoomgtm.com/marketing-quiz`
- `status` — `"published"` (live) or `"draft"` (hidden). Default: `"published"`
- `visibility` — `"public"` (listed) or `"unlisted"` (link-only). Default: `"unlisted"`

**Response (201):**
```json
{
  "ok": true,
  "data": {
    "catalog_id": "01HXY...",
    "slug": "marketing-quiz",
    "name": "Marketing Knowledge Quiz",
    "status": "published",
    "visibility": "public",
    "url": "https://yourname.catalogs.cloud.zoomgtm.com/marketing-quiz"
  }
}
```

### View a quiz

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id
```

Returns the full quiz including its schema.

### Update a quiz

```
PUT https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id
```

All fields are optional — only send what you want to change:

```json
{
  "name": "Updated Quiz Name",
  "schema": { ... },
  "status": "draft",
  "slug": "new-quiz-slug",
  "old_slug_action": "redirect"
}
```

When changing the slug, `old_slug_action` controls what happens to the old URL:
- `"redirect"` (default) — old URL redirects to the new one
- `"release"` — old URL becomes available for reuse

### Delete a quiz

```
DELETE https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id
```

---

## Analytics & Results

All analytics endpoints require authentication. Each analytics call costs **1 credit**. Event tracking (visitor activity) is **free**.

### Overview metrics

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id
```

**Query params:** `start`, `end` (ISO dates, e.g. `2024-01-01`)

Returns aggregate metrics: unique visitors, total page views, form submissions, conversion rate, page-level views, variant breakdown, referrer sources, checkout stats, and revenue.

### Timeseries (daily/hourly trends)

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/timeseries
```

**Query params (required):** `start`, `end` (ISO dates), `interval` (`day` or `hour`)

```json
{
  "ok": true,
  "data": [
    { "date": "2024-01-01", "page_views": 150, "sessions": 80, "form_submits": 25, "checkout_completes": 5, "revenue_cents": 4900 }
  ]
}
```

### Drop-off analysis

See exactly where visitors abandon your quiz:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/dropoff
```

**Query params:** `start`, `end` (ISO dates)

```json
{
  "ok": true,
  "data": {
    "total_visitors": 500,
    "pages": [
      { "page_id": "questions", "visitors": 500, "drop_off_rate": 0 },
      { "page_id": "results", "visitors": 350, "drop_off_rate": 30 }
    ],
    "fields": [
      { "field_id": "questions/email", "completions": 300, "completion_rate": 85.7 }
    ]
  }
}
```

### Answer breakdowns

See how visitors answered each question — great for understanding what resonates:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/responses
```

**Query params:** `start`, `end`, `page_id`, `component_id` (all optional)

```json
{
  "ok": true,
  "data": {
    "components": {
      "questions/q1": {
        "total_responses": 200,
        "distribution": {
          "Call To Action": { "count": 112, "percent": 56 },
          "Click To Act": { "count": 28, "percent": 14 },
          "Create The Ad": { "count": 60, "percent": 30 }
        }
      }
    }
  }
}
```

### Raw events

Browse individual visitor events with filtering:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/events
```

**Query params:** `start`, `end`, `cursor`, `limit` (default 100, max 500), `event_type`, `page_id`, `component_id`, `variant_slug`, `utm_source`, `utm_medium`, `utm_campaign`, `referrer`

Response includes a `cursor` for pagination (null when done).

### Visitor journey

Trace a single visitor's complete journey through your quiz, including their score:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/tracers/:tracerId
```

Returns every event in chronological order with a summary: total events, first/last seen, pages viewed, and whether they submitted.

---

## A/B Split Tests

Test different quiz versions to find what converts best. Split tests route visitors to different quiz variants based on weighted traffic distribution.

### Create a split test

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/split-tests
```

```json
{
  "slug": "marketing-quiz",
  "name": "Marketing Quiz A/B Test",
  "destinations": [
    { "slug": "marketing-quiz-v1", "weight": 50, "label": "Control" },
    { "slug": "marketing-quiz-v2", "weight": 50, "label": "Shorter version" }
  ]
}
```

The `slug` is the URL visitors see. They get routed to one of the destination quizzes based on weights. Visitors are sticky — they always see the same variant on return visits.

### Other split test operations

- `GET /api/v1/split-tests` — List all split tests
- `GET /api/v1/split-tests/:slug` — Get one split test
- `PUT /api/v1/split-tests/:slug` — Update (change `name`, `destinations`, or `status`: `"active"` / `"paused"`)
- `DELETE /api/v1/split-tests/:slug` — Delete a split test

---

## Schema Introspection

Get a map of all pages and components in a quiz — useful for understanding the structure before querying analytics:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id/schema/ids
```

```json
{
  "pages": {
    "questions": { "title": "Marketing Knowledge", "index": 0 },
    "results": { "title": "Your Results", "index": 1 }
  },
  "components": {
    "questions/q1": { "type": "multiple_choice", "label": "What does CTA stand for?", "quiz": true },
    "questions/email": { "type": "email", "label": "Your Email", "required": true }
  },
  "routing_entry": "questions"
}
```

---

## API Keys

Manage API keys for team members or integrations.

- `POST /api/v1/api-keys` — Create a key (roles: `reader`, `editor`, `admin`, `custom`). Returns the secret once — store it securely.
- `GET /api/v1/api-keys` — List all keys (secrets redacted)
- `DELETE /api/v1/api-keys/:keyId` — Revoke a key
- `POST /api/v1/api-keys/:keyId/rotate` — Rotate: revokes old key, creates new one with same config

---

## Videos

Upload video content to use in your quizzes with automatic HLS transcoding:

- `POST /api/v1/videos/upload` — Upload a video file
- `GET /api/v1/videos/:videoId/status` — Check transcoding progress
- `GET /api/v1/videos/:videoId/hls_url` — Get the playback URL

---

## Webhooks

If your quiz has a `webhook_url` configured in its schema, all visitor events are forwarded there in real time. Each webhook payload includes an `event_id` (ULID) for deduplication and `schema_ref` with human-readable page/component context.

---

## Variant Analytics

Every quiz gets an automatic `catalog:{catalog_id}` tag. To compare analytics across quiz variants, add the base quiz's `catalog:{base_id}` tag to each variant's `schema.tags`. API keys scoped with matching `tag_patterns` can then query analytics across all tagged variants.

---

## Quiz Schema Reference

A quiz is defined as a JSON schema with scored components. Here's a complete quiz example:

```json
{
  "slug": "marketing-quiz",
  "pages": [
    {
      "id": "questions",
      "title": "Marketing Knowledge",
      "components": [
        {
          "id": "q1",
          "type": "multiple_choice",
          "label": "What does CTA stand for?",
          "options": ["Click To Act", "Call To Action", "Create The Ad"],
          "quiz": { "correct_answer": "Call To Action", "points": 10, "explanation": "CTA = Call To Action" }
        },
        {
          "id": "q2",
          "type": "multiple_choice",
          "label": "What is a conversion rate?",
          "options": ["Bounce rate", "% of visitors who take desired action", "Click rate"],
          "quiz": { "correct_answer": "% of visitors who take desired action", "points": 10 }
        }
      ],
      "submit_label": "See Results"
    },
    {
      "id": "results",
      "title": "Your Results",
      "components": [
        { "id": "score-display", "type": "heading", "text": "Your Score" }
      ]
    }
  ],
  "routing": {
    "entry": "questions",
    "edges": [
      { "from": "questions", "to": "results", "is_default": true }
    ]
  }
}
```

### Quiz Scoring

Add a `quiz` property to any component to make it scored:

```json
{
  "id": "q1",
  "type": "multiple_choice",
  "label": "What does CTA stand for?",
  "options": ["Click To Act", "Call To Action", "Create The Ad"],
  "quiz": { "correct_answer": "Call To Action", "points": 10, "explanation": "CTA = Call To Action" }
}
```

The runtime automatically tracks: `total` (earned points), `max` (possible points), `percent` (score %), `correct_count`.

### Score-Based Routing

Route visitors to different results pages based on their quiz score:

```json
{
  "from": "questions",
  "to": "high-score-page",
  "conditions": {
    "match": "all",
    "rules": [{ "score": "percent", "operator": "greater_than", "value": 80 }]
  }
}
```

### Component Types (56 total)

**Input (27):** `short_text`, `long_text`, `rich_text`, `email`, `phone`, `url`, `password`, `number`, `currency`, `date`, `datetime`, `time`, `date_range`, `dropdown`, `multiselect`, `multiple_choice`, `checkboxes`, `picture_choice`, `star_rating`, `slider`, `file_upload`, `signature`, `address`, `location`, `switch`, `checkbox`, `choice_matrix`, `ranking`, `opinion_scale`

**Display (11):** `heading`, `paragraph`, `banner`, `image`, `video`, `pdf_viewer`, `social_links`, `html`, `divider`, `faq`, `testimonial`, `pricing_card`

**Layout (3):** `section_collapse`, `table`, `subform`

### Popups

Trigger popups based on visitor behavior:

```json
{
  "popups": [
    {
      "id": "exit-popup",
      "trigger": { "type": "exit_intent", "delay_ms": 3000 },
      "pages": ["questions"],
      "mode": "modal",
      "content": { "title": "Almost done!", "body": "Finish the quiz to see your results" }
    }
  ]
}
```

**Trigger types:** `exit_intent`, `scroll_depth`, `inactive`, `timed`, `page_count`, `custom`, `video_progress`, `video_chapter`

### Completion Screen

Customize what visitors see after submitting the quiz:

```json
{
  "settings": {
    "completion": {
      "heading": "Thanks for taking the quiz!",
      "message": "Check your email for your results.",
      "redirect_url": "https://example.com",
      "redirect_delay": 3000,
      "actions": [
        { "type": "fill_again", "label": "Retake Quiz", "style": "secondary" },
        { "type": "share", "label": "Share", "style": "ghost" },
        { "type": "redirect", "label": "Visit Site", "url": "https://example.com", "style": "primary" }
      ]
    }
  }
}
```

**Action types:** `fill_again` (reset form), `share` (copy URL), `redirect` (navigate to URL). All fields are optional — omit `completion` entirely for a minimal checkmark screen.

### Condition Operators

`equals`, `not_equals`, `contains`, `not_contains`, `greater_than`, `less_than`, `greater_than_or_equal`, `less_than_or_equal`, `starts_with`, `ends_with`, `regex`, `in`, `not_in`, `is_empty`, `is_not_empty`, `between`

---

## CLI

Manage quizzes from the command line:

```bash
npx catalogs catalog push quiz-schema.json --publish    # Create or update a quiz from a JSON file
npx catalogs catalog list                                # List all your quizzes
npx catalogs video upload ./intro.mp4                    # Upload a video
npx catalogs video status VIDEO_ID                       # Check transcoding progress
```

---

## AI Variant Routing

Automatically route visitors to the best quiz variant using natural language hints. Instead of requiring exact variant slugs, pass a description and let the AI pick the right variant.

### Route a visitor with hints

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/public/route-variant?subdomain=yourname&slug=marketing-quiz&hints=female+entrepreneur+interested+in+social+media
```

**Response:**
```json
{
  "ok": true,
  "data": {
    "variant_slug": "problem-aware-female",
    "reason": "ai_matched"
  }
}
```

`reason` values: `ai_matched` (LLM picked best match), `single_variant` (only one variant exists), `no_variants` (quiz has no variants), `fallback` (LLM couldn't decide, returned first variant).

### Frontend hint URLs

The frontend handles AI routing automatically — just add `hints` to the URL:

```
# AI-routed (base quiz shows immediately, variant hot-swaps when AI responds):
https://yourname.catalogs.cloud.zoomgtm.com/marketing-quiz?hints=female+entrepreneur&ref=253

# Silent redirect (for affiliates — suppresses event tracking):
https://yourname.catalogs.cloud.zoomgtm.com/marketing-quiz?hints=problem+aware+male&silent_redirect=true&ref=253

# After AI routing resolves, browser URL updates to:
https://yourname.catalogs.cloud.zoomgtm.com/marketing-quiz/problem-aware-male?ref=253
```

The base quiz renders instantly while AI routing resolves in the background. Visitors never see a loading screen — the variant swap is seamless.

---

## Sandbox Mode

Edit quizzes safely without affecting production. A sandbox is a full clone of your quiz with its own URL and schema — make changes, preview live, and promote when ready.

### Create a sandbox

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id/sandbox
```

```json
{
  "suffix": "redesign-v2"
}
```

**Response (201):**
```json
{
  "ok": true,
  "data": {
    "catalog_id": "01ABC...",
    "slug": "marketing-quiz--redesign-v2",
    "name": "Marketing Knowledge Quiz (Sandbox: redesign-v2)",
    "sandbox_of": "01HXY...",
    "parent_slug": "marketing-quiz",
    "url": "https://yourname.catalogs.cloud.zoomgtm.com/marketing-quiz--redesign-v2"
  }
}
```

The sandbox is a regular quiz with its own URL. Edit it freely using `PUT /api/v1/catalogs/:sandbox_id` — your production quiz is untouched. The frontend shows an amber "SANDBOX" banner so you always know you're in sandbox mode.

### List sandboxes for a quiz

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id/sandboxes
```

### Promote sandbox to production

Copy the sandbox schema to the parent quiz:

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:sandbox_id/promote
```

```json
{
  "delete_sandbox": true
}
```

By default the sandbox is deleted after promotion. Set `"delete_sandbox": false` to keep it.

### Discard a sandbox

```
DELETE https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:sandbox_id
```

### Listing quizzes with sandboxes

By default, `GET /api/v1/catalogs` hides sandboxes. Add `?include_sandboxes=true` to include them. Each quiz response includes `sandbox_of` (null for regular quizzes, parent quiz ID for sandboxes).

---

## Element Inspector (DevEx)

Built-in developer tool for AI agent workflows. Hold **Shift+Alt** and hover over any element in a live quiz to see its exact reference path (`pageId/componentId`) with a one-click copy button.

This makes it trivial to tell an AI agent exactly which element to modify — no guessing, no digging through JSON.

The reference format matches the schema introspection endpoint (`GET /api/v1/catalogs/:id/schema/ids`), so copied references map 1:1 to API paths.

**How to use:**

1. Open any quiz in the browser
2. Hold **Shift+Alt** — a "Inspector active" indicator appears
3. Hover over any element — it highlights with an indigo border and shows a tooltip with `pageId/componentId (type)`
4. Click **Copy** to copy the reference to clipboard
5. Paste the reference into your AI agent conversation (e.g. "change the heading at `questions/hero-title`")

---

## Event Tracking (Free)

Visitor events are tracked automatically by the quiz frontend. You can also send custom events:

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/events
```

**Valid event types:** `page_view`, `field_change`, `field_complete`, `form_submit`, `action_click`, `exit_intent`, `session_start`, `session_resume`, `cart_add`, `cart_remove`, `checkout_start`, `checkout_skip`, `checkout_complete`, `payment_info_added`, `offer_declined`, `lead_captured`, `video_play`, `video_pause`, `video_progress`, `video_complete`, `video_chapter`, `video_seek`

Batch up to 25 events: `POST /events/batch` with `{ "events": [...] }`
