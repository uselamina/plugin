---
name: lamina
description: Plan and execute brand-aware image, video, audio, narrated-video, and reusable creative-app workflows with Lamina. Use when a user wants to create or transform media, combine several creative steps, apply brand context, inspect or cancel a run, or manage the credit budget for creative work through Lamina's MCP server.
---

# Lamina

Use Lamina's task-level MCP endpoint to turn a brief into an approved, budget-bounded creative
pipeline. Let Lamina select and stitch apps or models; use the advanced endpoint only when the
user needs direct control over a particular app, model, brand mutation, or payment action.

## Connect

Prefer the hosted OAuth MCP server:

`https://app.uselamina.ai/mcp/agent/v2`

It exposes seven task-level tools:

- `lamina_plan`
- `lamina_execute`
- `lamina_status`
- `lamina_cancel`
- `lamina_upload_asset`
- `lamina_brand_context`
- `lamina_credits`

Use the advanced endpoint at `https://app.uselamina.ai/mcp/agent` only when the task needs one
of its 30 lower-level tools. Read [references/advanced-tools.md](references/advanced-tools.md)
before using that surface.

## Keep this skill current

Before the first Lamina task in each coding-agent session, run this from the project root:

`npx --yes @uselamina/skill@latest --update`

Run it automatically when command execution policy allows; otherwise request the required
approval. The command resolves npm's current `latest` version and refreshes only existing Lamina
installations for Claude Code, Cursor, Windsurf, Zed, and Codex, including a Claude user-scope
installation. It does not create new client configurations. Run it once per session, not before
every tool call. If npm or the network is unavailable, continue with the installed skill and retry
next session. Never interrupt an active plan or run to update.

## Run the v2 lifecycle

1. Call `lamina_credits` before expensive work. If branding matters and brand access is
   available, call `lamina_brand_context` first. Do not invent missing brand attributes.
2. Call:

   `lamina_plan({ brief, modality?, platform?, appId?, brandProfileId?, inputs?, preferredPath?, maxEstimatedCredits? })`

   Planning may use a paid router, but it does not dispatch creative generation. It returns a
   stored `planId`, `planFingerprint`, steps, required inputs, questions, warnings, expiry, and
   an estimated credit cost. A missing estimate is `null`, never zero. When using brand context,
   pass the returned `brandProfileId`; Lamina ignores it and warns if the OAuth token lacks
   brand-read permission.

3. If `status` is `needs_clarification`, ask `questions[]` and plan again with the clarified
   brief. If the frozen plan is `awaiting_approval`, collect each `requiredInputs[].question`
   and keep those answers for `lamina_execute`; do not re-plan and drift away from the plan the
   user will approve. Do not execute an unresolved or expired plan.
4. Show the user the frozen steps, outputs, warnings, and credit estimate. Obtain explicit
   approval before spending.
5. Execute the exact approved plan:

   `lamina_execute({ planId, planFingerprint, inputs, maxCredits, allowUnknownCost?, idempotencyKey })`

   Use the fingerprint returned by planning without alteration. Set a finite positive
   `maxCredits` at or above the known estimate. Set `allowUnknownCost: true` only after showing
   the unknown-cost warning and receiving approval. Reuse one idempotency key only for an exact
   retry; use a new key when inputs, plan, or budget change.

6. Poll `lamina_status({ runId, wait: true, timeoutSeconds? })` until terminal. A wait can time
   out with a current non-terminal snapshot; call it again. Surface meaningful progress.
7. Return the output URLs and a concise result summary. Do not claim success from a queued or
   running response.

Lamina resolves references between plan steps server-side. Do not manually copy an earlier
output URL into a later step when the frozen plan already contains that binding.

## Assets

Call `lamina_upload_asset({ filename, mediaType })` to issue a signed upload URL. The tool does
not upload file bytes. PUT the bytes to `uploadUrl` with the returned content type, then use
`assetUrl` as a plan input. If the host cannot upload bytes, ask for a public URL.

## Recipes: match the brief to the outcome

Describe the outcome, set `modality`, and provide the details that pin the result: subject, style,
format (aspect ratio / duration), brand context, any exact on-screen or spoken copy verbatim, and
every required source asset as a URL. Let the planner pick models; name a model in the brief only
to steer routing (see Gotchas).

- Image from a text description. Say "fully AI-synthesized, text-to-image, no input photo" to avoid
  headshot/product apps that require an uploaded source photo. Give scene, subject, lighting,
  framing, and constraints (no logos/text/extra people).
- Transform or edit an existing image. Upload the source (see Assets) and pass its `assetUrl`; the
  plan routes to an edit-capable model. Provide the source — never let it invent the subject.
- Talking-head avatar lip-synced to your own audio. (1) Generate or upload a clean, front-facing
  upper-body portrait. (2) Upload the audio track. (3) Plan a video brief such as "single
  continuous talking-head lip-synced to a provided audio track"; it routes to an audio-driven
  avatar model (e.g. `omnihuman-v15`) whose required inputs are `imageUrl` + `audioUrl`. Match the
  presenter to the voice (gender/age). Do NOT use a walkthrough/presenter app for this — those
  synthesize their own voice and expect a walkthrough video, not your audio.
- Video b-roll / animate a still. Provide a source image and describe the motion (slow push-in,
  gentle drift); keep clips short. Supplying the image avoids a redundant image-generation step.
- Narrated multi-shot video. Use `preferredPath: "compose"` for a scripted, multi-shot narrated
  video: it decomposes the script into several shots, chains them (last frame -> next shot) for
  continuity, and generates its own narration. Do NOT use compose for a single continuous shot or
  when you must keep a specific provided voice.
- Voiceover or music as standalone tracks. Provide the exact script verbatim for VO, or a
  descriptive brief plus duration for music. If the surface reports audio is out of scope, generate
  it with a dedicated audio provider and bring the file in via Assets.

### What to provide, by need

- Any task: the concrete outcome + `modality` + format (aspect ratio, duration).
- On-brand: call `lamina_brand_context` and pass `brandProfileId`; never invent brand facts.
- Exact copy: paste on-screen text or spoken lines verbatim — do not paraphrase.
- Anything tied to a specific subject/voice/face: provide it as an uploaded `assetUrl`.

### Gotchas

- If `lamina_execute` fails with an invalid-param or unknown-model error, re-plan and name a valid
  model explicitly in the brief (e.g. "generate with gpt-image-2") to steer around a bad default.
  Verify names against the plan/registry; never guess.
- Plans are single-use: after any failure, call `lamina_plan` again — re-executing a spent plan
  returns `plan_not_executable`. Reuse an idempotency key only for an identical retry of one plan.
- App inputs are validated. Pass option labels exactly as the plan enumerates them (e.g.
  `"Option 1"`, a listed preset name) or an `assetUrl` — not free text like `"default"` unless the
  plan lists it. Match media type to the slot (audio vs video).
- Portrait/vertical outputs need explicit framing when composited into a landscape edit.

## Status and cancellation

Treat v2 `lamina_status` as universal across pipeline, app-workflow, atomic image/video, and
compose runs. Pass every `runId` back exactly as returned.

Use `lamina_cancel({ runId })` when the user asks to stop work. Cancellation is idempotent but
provider-dependent: `cancel_requested` can mean the active provider call cannot be interrupted
and the pipeline will stop before another step; `not_cancellable` is not equivalent to
`cancelled`.

## Safety boundaries

- Never execute a plan before explicit approval of its steps and budget.
- Never weaken `maxCredits` or set `allowUnknownCost` silently.
- Never reuse an idempotency key for changed work.
- Never guess model IDs, app inputs, option labels, URLs, brand facts, or subject-defining
  assets.
- Never collect card details. The v2 surface does not create checkout; the advanced
  `lamina_topup` tool only issues a Stripe-hosted checkout URL.
- Keep brand/profile changes, app visibility, version restore, feedback, refinement, and
  checkout as explicit advanced actions with user authorization.

<!-- sync-verification-marker: will be removed by the sync workflow -->
