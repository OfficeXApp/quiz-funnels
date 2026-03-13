---
name: quiz-funnels
description: |
  REST API client for the Quiz Funnels app on OfficeX. Declarative quiz and funnel builder with scoring, conditional routing, intent-based personalization, video tracking, and Stripe checkout.
  Use when: (1) Creating or managing quiz/funnel schemas via API, (2) Pushing quiz JSON configs, (3) Querying funnel analytics and visitor journeys, (4) Managing API keys, (5) Uploading and transcoding videos for quizzes, (6) Creating Stripe checkout sessions.
  Triggers: quiz funnel, quiz builder, quiz scoring, quiz api, lead quiz, personality quiz, assessment quiz, funnel builder, conversion quiz, interactive quiz
---

# Quiz Funnels -- API Skill

Declarative quiz and funnel builder SaaS. Define quizzes, assessments, and multi-step funnels as JSON/TypeScript schemas with 56+ component types, quiz scoring, conditional routing, intent-based personalization, popup triggers, video tracking, cart/checkout, and analytics. AI agents use this API to CRUD quiz funnels and query conversion data.

> **Install on OfficeX:** [officex.app/store/en/app/quiz-funnels](https://officex.app/store/en/app/quiz-funnels)

## Prerequisites

After installing the app on OfficeX, you receive credentials via the install webhook. Or self-signup at the frontend and create API keys from Settings.

```bash
# Option A: OfficeX install credentials
OFFICEX_INSTALL_ID="your_install_id"
OFFICEX_INSTALL_SECRET="your_install_secret"

# Option B: API key (created from Settings page)
CF_API_KEY="cfk_..."

# API base URL
CF_API_URL="https://catalog-funnel-api.cloud.zoomgtm.com"
```

## Authentication

**OfficeX credentials:** Bearer token = Base64(install_id:install_secret)

```bash
TOKEN=$(echo -n "${OFFICEX_INSTALL_ID}:${OFFICEX_INSTALL_SECRET}" | base64)
curl -H "Authorization: Bearer $TOKEN" $CF_API_URL/api/v1/catalogs
```

**API key:** Pass the `cfk_...` token directly as Bearer.

```bash
curl -H "Authorization: Bearer cfk_..." $CF_API_URL/api/v1/catalogs
```

## API Reference

### Base URLs

| Stage | API URL |
|---|---|
| Production | `https://catalog-funnel-api.cloud.zoomgtm.com` |
| Staging | `https://catalog-funnel-api-staging.cloud.zoomgtm.com` |

Frontend: `https://{subdomain}.catalogs.cloud.zoomgtm.com/{quiz-slug}`

### GET /health

Health check (unauthenticated).

```json
{ "ok": true, "stage": "production" }
```

---

### Catalogs (CRUD)

All catalog endpoints require authentication. Quizzes are catalogs with quiz-scored components.

#### GET /api/v1/catalogs

List all catalogs/quizzes for the authenticated user.

**Response:**
```json
{
  "ok": true,
  "data": [
    {
      "catalog_id": "01HXY...",
      "slug": "marketing-quiz",
      "name": "Marketing Quiz",
      "status": "published",
      "visibility": "public",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

#### POST /api/v1/catalogs

Create a new quiz funnel.

**Request Body:**
```json
{
  "slug": "marketing-quiz",
  "name": "Marketing Knowledge Quiz",
  "schema": { ... },
  "status": "published",
  "visibility": "public"
}
```

- `slug`: lowercase alphanumeric with hyphens
- `schema`: CatalogSchema object (see Schema section below)
- `status`: `"published"` | `"draft"` (default: `"published"`)
- `visibility`: `"public"` | `"unlisted"` (default: `"unlisted"`)

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

#### GET /api/v1/catalogs/:id

Get a single quiz with full schema.

#### PUT /api/v1/catalogs/:id

Update a quiz. Supports partial updates. Slug changes can create redirects.

**Request Body (all fields optional):**
```json
{
  "slug": "new-slug",
  "name": "Updated Name",
  "schema": { ... },
  "status": "draft",
  "visibility": "public",
  "old_slug_action": "redirect"
}
```

#### DELETE /api/v1/catalogs/:id

Delete a quiz.

---

### Public Catalog Fetch (No Auth)

#### GET /public/catalogs/:subdomain/:slug

Fetch a published quiz schema for rendering.

---

### Analytics

All analytics endpoints require authentication with `analytics:read` permission.

#### GET /api/v1/analytics/catalogs/:id

Funnel metrics including quiz completion rates, page drop-off, field completions, variant breakdown, and referrer sources.

**Query params:** `start`, `end` (ISO dates), `limit` (max events)

#### GET /api/v1/analytics/catalogs/:id/events

Raw events for a quiz funnel.

#### GET /api/v1/analytics/tracers/:tracerId

Full visitor journey for a tracer (visitor) ID.

---

### Event Tracking (No Auth)

#### POST /events

Track a single event.

**Valid event_type values:** `page_view`, `field_change`, `field_complete`, `form_submit`, `action_click`, `exit_intent`, `session_start`, `session_resume`, `cart_add`, `cart_remove`, `checkout_start`, `checkout_skip`, `video_play`, `video_pause`, `video_progress`, `video_complete`, `video_chapter`, `video_seek`

#### POST /events/batch

Track up to 25 events at once. Body: `{ "events": [...] }`

---

### API Keys

#### POST /api/v1/api-keys

Create an API key. Roles: `reader`, `editor`, `admin`, `custom`.

#### GET /api/v1/api-keys

List all keys (secrets redacted).

#### DELETE /api/v1/api-keys/:keyId

Revoke a key.

#### POST /api/v1/api-keys/:keyId/rotate

Rotate: revoke old + create new with same config.

---

### Videos

#### POST /api/v1/videos/upload -- Upload video for HLS transcoding
#### GET /api/v1/videos/:videoId/status -- Check transcode status
#### GET /api/v1/videos/:videoId/hls_url -- Get HLS playback URL

---

## Quiz Schema Reference

A quiz is a CatalogSchema with quiz-scored components. Minimal quiz example:

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

Components with `quiz` config are scored automatically. The runtime tracks:
- `total`: sum of earned points
- `max`: sum of possible points
- `percent`: (total / max) * 100
- `correct_count`: number of correct answers

### Score-Based Routing

Route visitors based on quiz scores:

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

### Condition Operators

`equals`, `not_equals`, `contains`, `not_contains`, `greater_than`, `less_than`, `greater_than_or_equal`, `less_than_or_equal`, `starts_with`, `ends_with`, `regex`, `in`, `not_in`, `is_empty`, `is_not_empty`, `between`

### CLI Usage

```bash
npx catalogs catalog push quiz-schema.json --publish
npx catalogs catalog list
npx catalogs video upload ./intro.mp4
npx catalogs video status VIDEO_ID
```
