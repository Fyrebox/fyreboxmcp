---
name: fyrebox-quiz-maker
description: >-
  Create, edit, and publish interactive quizzes, assessments, personality
  quizzes, and branching scenario games with Fyrebox, directly from chat. Use
  this skill whenever the user wants to build a quiz, make a test or assessment,
  create a personality or "which X are you" quiz, set up a lead-generation or
  product-recommendation quiz, add branching/scenario logic, design quiz
  appearance, generate an AI background image for a quiz, capture leads from a
  quiz, or review quiz responses. Triggers include "make a quiz", "build an
  assessment", "create a personality quiz", "quiz that captures emails",
  "scenario quiz", "recommend a product based on answers", or any request to
  create or manage Fyrebox quizzes. Requires the Fyrebox MCP connector to be
  connected.
---

# Fyrebox Quiz Maker

Fyrebox turns a chat request into a real, shareable interactive quiz. You build
the quiz with the connector's tools, and every quiz publishes to a public
`/play/<slug>` URL the user can share or embed on any website.

## Before you start

The Fyrebox MCP connector must be connected (OAuth 2.0). On first use, call
`whoami` to confirm the account is authenticated and to see the plan and how
many AI image credits are available. If a tool returns a `FORBIDDEN` scope
error, the user's connection didn't grant that capability (e.g. reading leads
requires the `leads:read` scope, granted only on explicit request).

## The four quiz types

Every quiz has one `type`:

- **trivia** — right/wrong questions; each question needs one `correct` option.
- **assessment** — scored test with a passing threshold and pass/fail results.
- **personality** — options carry `score` toward outcomes (e.g. "which X are you").
- **scenario** — branching quiz where each answer routes to another question or
  to an ending (outcome). Use this for product recommendations, "choose your
  path" experiences, and decision flows.

## Core workflow

1. **Clarify intent** — type of quiz, topic, roughly how many questions, and
   whether it should collect leads (emails). Don't over-ask; sensible defaults
   are fine.
2. **Create** — `create_quiz` (or `create_scenario_quiz` for branching, or
   `apply_template` to start from a ready-made template).
3. **Refine** — adjust questions, appearance, lead capture, and results.
4. **Hand back the share URL** returned by the create/get tools so the user can
   play or embed it.

## Editing pattern (important)

To change an existing quiz's content reliably:

1. Call `get_quiz` with the `gid` — it returns an editable `spec` (questions,
   options, correct answers, scenario branching, endings) plus design fields.
2. Edit that `spec` in place.
3. Pass it back via `update_quiz` as the `patch`. If `questions` is included,
   the **entire** question set is replaced, so send the full list, not a
   fragment.

## Tool reference

**Account & discovery**
- `ping` — health check; confirms the server is reachable.
- `whoami` — authenticated account: name, email, plan, remaining AI credits.
- `list_quizzes` — all quizzes for the account, newest first.
- `get_quiz` — fetch one quiz by `gid` as an editable spec + design.
- `list_templates` — ready-made templates to instantiate.

**Create & manage**
- `create_quiz` — new quiz of any supported type. Provide `questions` as a
  nested list of `{text, options:[{text, correct?, score?, next?}]}`.
- `create_scenario_quiz` — branching quiz; set each option's `next` to
  `{question:<index>}` or `{ending:<index>}` and provide at least one ending.
- `apply_template` — create a new quiz from a template (copies content, design,
  rules, and scoring categories).
- `clone_quiz` — duplicate a quiz into a new one with a new id.
- `update_quiz` — patch `name`, `type`, `locale`, `instructions`, `questions`,
  or `endings`.
- `delete_quiz` — permanently delete a quiz and all its leads/results.
  Irreversible — confirm with the user first.

**Scenario branching & endings**
- `set_branching` — route specific answers; each branch references a question
  and option by 0-based index with `next={question:<index>}` or `{ending:<index>}`.
- `add_ending` / `update_ending` / `delete_ending` — manage outcome screens
  (label, message, background, redirect). Endings are addressed by 0-based index.
- `set_ending_rule` — advanced per-ending behavior: custom message, background,
  redirect, lead-capture fields, certificate, or a custom result page.

**Appearance & media**
- `customize_appearance` — background color/image, text color/font/size, accent
  color, button styling, and logo.
- `set_background_image` — set the background to an existing image URL.
- `generate_background_image` — generate an AI background from the quiz's
  content (or a `customPrompt`) and set it. Uses one AI image credit.

**Leads & results**
- `set_lead_capture` — configure the lead form: which fields to collect on a
  pass vs a fail, whether the form is optional, and admin notifications.
- `set_result_pages` — configure pass/fail result screens: outcome message,
  optional custom HTML page, redirect URL, and the passing score threshold.
- `get_leads` — fetch responses for a quiz (name, email, score, outcome,
  answers), newest first. Requires the `leads:read` scope.

## Examples

- **Lead generation:** "Create a 5-question quiz 'Which CRM is right for you?'
  that asks for the player's email before showing a personalized result."
  → `create_quiz` (type personality), then `set_lead_capture`, then share the URL.
- **Graded assessment:** "Build a 10-question biology test for my Grade 9 class
  with a 70% pass mark and custom pass/fail pages."
  → `create_quiz` (type assessment), then `set_result_pages` with a 70 threshold.
- **Product recommendation:** "Make a branching quiz that recommends one of our
  three skincare bundles based on answers."
  → `create_scenario_quiz` with three endings and `next` routing per option.
- **Branding:** "Apply our brand colors and generate an on-theme background for
  my quiz, then give me the link."
  → `customize_appearance`, then `generate_background_image`, then `get_quiz`.

## Tips & gotchas

- Always return the share/play URL after creating or meaningfully changing a quiz.
- Endings, questions, and branch options are **0-based indexed**; double-check
  indices when wiring scenario routes.
- `update_quiz` replaces the full question set when `questions` is supplied —
  fetch with `get_quiz`, edit the spec, and send it back whole.
- `generate_background_image` consumes an AI credit; check `whoami` if a user is
  on a limited plan and avoid regenerating unnecessarily.
- `delete_quiz` and `delete_ending` are destructive — confirm before calling.
