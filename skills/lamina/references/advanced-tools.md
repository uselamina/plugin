# Lamina advanced MCP tools

Use this reference only with `https://app.uselamina.ai/mcp/agent`. The endpoint exposes the 30
tools below for direct control. Prefer the seven-tool v2 endpoint for ordinary requests and
multi-step pipelines.

## Contents

- [Rules shared by all tools](#rules-shared-by-all-tools)
- [Discovery, apps, and lifecycle](#discovery-apps-and-lifecycle)
- [Atomic generation](#atomic-generation)
- [Narrated video composition](#narrated-video-composition)
- [Brand and brand kit](#brand-and-brand-kit)
- [Generated-app management](#generated-app-management)
- [Credits and checkout](#credits-and-checkout)
- [Reliable cross-tool workflows](#reliable-cross-tool-workflows)

## Rules shared by all tools

- Treat every paid dispatch or mutation as a side effect. Show the important inputs and obtain
  approval before calling it.
- Under OAuth, the workspace comes from the authorized identity. Do not request or paste API
  keys. Compatibility clients without OAuth may expose an `apiKey` argument.
- Check `lamina_credits` before expensive generation. An absent estimate means unknown, not
  free.
- Pass IDs and output URLs exactly as returned. Never invent app IDs, model IDs, parameter
  keys, option labels, or asset URLs.
- The advanced `lamina_status` accepts atomic, workflow, compose, and pipeline run IDs. Legacy
  compose polling through `lamina_compose_video({ runId, wait? })` remains supported.
- `lamina_cancel` is universal, workspace-scoped, and idempotent. A truthful
  `cancel_requested` or `not_cancellable` response is not the same as confirmed cancellation.

## Discovery, apps, and lifecycle

| Tool                  | Main inputs                                                            | Use and lifecycle                                                                                                                                                                                                                                                                                        |
| --------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lamina_discover`     | `keywords` (1–10 strings), `limit?`                                    | Find apps by creative purpose. Rephrase and retry if needed; then describe a candidate. An omitted `estimatedCredits` is unknown.                                                                                                                                                                        |
| `lamina_describe`     | `appId`                                                                | Read exact parameter keys, types, options, defaults, outputs, and estimated cost. Ask about user-owned assets and hidden preset groups instead of silently accepting demo defaults.                                                                                                                      |
| `lamina_run`          | `appId`, `inputs`; `outputs?`, `applyBrand?`, `brandProfileId?`        | Dispatch an authorized app run. Key inputs by the described stable parameter `key`; output subsets use described labels. Returns a workflow `runId` for status.                                                                                                                                          |
| `lamina_status`       | `runId`; `wait?`, `timeoutSeconds?`                                    | Read or wait for a pipeline, workflow, atomic, or compose run. Every family returns the same normalized lifecycle and completed assets in `outputs[]`. A timed-out wait returns the latest snapshot; poll again.                                                                                         |
| `lamina_upload_asset` | `filename`, `mediaType`                                                | Issue `uploadUrl`, `assetUrl`, and a content-type hint. It does not move bytes; PUT the local bytes to the signed URL before using `assetUrl`.                                                                                                                                                           |
| `lamina_cancel`       | `runId`                                                                | Request cancellation for any supported run family. Respect `cancel_requested` and `not_cancellable`; do not report cancellation unless the returned state confirms it.                                                                                                                                   |
| `lamina_create`       | `brief`; `modality?`, `platform?`, `appId?`, `inputs?`, `numVariants?` | **Plan only; never dispatches generation.** Branch on bare `status`: `plan`, `needs_clarification`, or `unmatched`. For `plan`, ask every `askUser` question and call `lamina_run` with `selectedApp.appId`, merged inputs, and selected output labels. Re-call create only after `needs_clarification`. |

### `lamina_create` handoff

For `status: "plan"`, preserve the selected plan:

1. Ask all `askUser[].question` items.
2. Merge answers into `draftedInputs` using each question's `name`.
3. Treat the special `__outputs` answer as output labels rather than an input.
4. Call `lamina_run` once. Do not re-run the planner and risk choosing a different app.

For `status: "unmatched"` with a `suggestion`, obtain approval before creating the bespoke app
with `lamina_generate_workflow`, then run it. Without a suggestion, explain that the brief is
outside the supported creative surface.

## Atomic generation

| Tool                     | Main inputs                                                                                                                                  | Use and lifecycle                                                                                                                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lamina_models_list`     | `modality?` (`image`, `video`, or `audio`)                                                                                                   | List curated model IDs with purpose, cost tier, speed, and provider. Prefer the cheapest suitable model; do not guess an ID.                                                                             |
| `lamina_models_describe` | `modelId`                                                                                                                                    | Read the chosen model's `paramSchema`, including required fields, allowed values, defaults, and cross-field rules.                                                                                       |
| `lamina_generate_image`  | `model`; `prompt?`, `params?`, `webhookUrl?`                                                                                                 | Dispatch one text-to-image or image-edit operation. Supply only fields declared by describe; source URLs select edit modes where supported. Returns an atomic `runId` for status.                        |
| `lamina_generate_video`  | `model`; `prompt?`, `params?`, `webhookUrl?`, `includeCitationKit?`                                                                          | Dispatch one video operation. Required image, video, reference, or keyframe URLs depend on the described model. `includeCitationKit` is a preview signal and currently returns `coming_soon`, not a kit. |
| `lamina_generate_audio`  | `model`; `prompt`, `params?`                                                                                                                 | Dispatch one audio operation — a voiceover (TTS models: `prompt` is the words to speak, `params.voiceId` selects the voice, incl. a brand-kit voice) or a music bed (`eleven_music`: `prompt` describes the music, `params.duration` sets length). Returns an atomic `runId` for status. |
| `lamina_plv_background`  | `product.category`; `product` details, `theme?`, `aspectRatio?`, `polish?`, `brandProfileId?`, `model?`, `webhookUrl?`                       | Generate a background-only Meta PLV plate with no product, people, or text. Poll the returned atomic run.                                                                                                |
| `lamina_generate_hook`   | `product.category`; `product.vertical?`, `theme?`, `aspectRatio?`, `durationSeconds?`, `polish?`, `brandProfileId?`, `model?`, `webhookUrl?` | Generate a short, product-free opener clip for an ad. Poll the returned atomic run.                                                                                                                      |

Always follow:

`lamina_models_list` → `lamina_models_describe` → approval → atomic dispatch →
`lamina_status`

Do not send model-specific fields that are absent from `paramSchema`. Structured validation
errors identify fields to correct; they are not authorization to switch models silently.

## Narrated video composition

| Tool                     | Main inputs                                                                                                      | Use and lifecycle                                                                                                                                                                                      |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `lamina_compose_formats` | `durationSeconds?`                                                                                               | List available narrated-video formats, required assets, capabilities, and estimated cost. An `available: false` format cannot be dispatched.                                                           |
| `lamina_compose_plan`    | `script`; `generationBrief?`, `direction?`, `sections?`, `format?`, `inputs?`, `aspectRatio?`, `brandProfileId?` | Preview routing, storyboard, missing inputs, and estimated credits. It does not generate media, though planning can use an LLM.                                                                        |
| `lamina_compose_video`   | Start with `script` plus optional directing fields; poll with `runId` and `wait?`                                | Dispatch a narrated multi-shot video, or poll an existing compose run. Never pass both `script` and `runId`. Prefer universal `lamina_status` after dispatch; the polling overload remains compatible. |

Before composition, draft spoken narration, show it with the format and aspect ratio, run
`lamina_compose_plan`, and obtain approval for the storyboard and cost. Use
`lamina_generate_video` instead when the user wants one short clip rather than a narrated
multi-shot production.

## Brand and brand kit

| Tool                           | Main inputs                                                                                           | Use, limits, and side effects                                                                                                                                                                                                                 |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lamina_brand`                 | Optional brand, campaign, workflow, platform, objective, modality, and `topK` filters                 | Read brand DNA, active guidance, and performance patterns. Empty sections mean unconfigured data; never fabricate it.                                                                                                                         |
| `lamina_set_brand`             | `brandName`; optional profile and brand attributes                                                    | Create or update a brand and starter DNA. Owner/admin mutation; obtain approval.                                                                                                                                                              |
| `lamina_brand_compliance`      | workflow `runId`                                                                                      | Read Brand Guard data already stored on a workflow execution. It is not an on-demand scorer and does not accept arbitrary atomic, compose, or pipeline IDs.                                                                                   |
| `lamina_brand_score`           | completed image-producing `runId`; `brandProfileId?`                                                  | Score a completed **image** output from a workflow, atomic, or pipeline run. Workflow runs can inherit the app brand; other run families require `brandProfileId`. Video-only, compose, or output-less runs return a typed unsupported error. |
| `lamina_brand_feedback`        | workflow `runId`, `verdict`; `note?`, `brandProfileId?`                                               | Record approve/reject feedback. A rejection note can mutate brand guardrails. Owner/admin only; ask explicitly before applying it.                                                                                                            |
| `lamina_refine_to_brand`       | completed workflow `runId`; `minScore?`, `maxIterations?`, `maxCredits?`, `brandProfileId?`           | Paid bounded refinement for a rerunnable workflow image output. It does not refine arbitrary atomic, compose, pipeline, video-only, or text-only runs. Keep the hard credit ceiling.                                                          |
| `lamina_brand_kit`             | `brandProfileId`                                                                                      | Read resolved voices, avatars, characters, motion graphics, and raw element rows. Empty categories mean none are registered.                                                                                                                  |
| `lamina_set_brand_kit_element` | `brandProfileId`, `elementType`, `name`; `refKind?`, `refId?`, `spec?`                                | Register or supersede a stable brand-kit handle. Owner/admin mutation. Element types are `voice`, `avatar`, `character`, and `motion_graphic`.                                                                                                |
| `lamina_save_to_brand_kit`     | `brandProfileId`, `elementType`, `name`, exactly one of `runId` or `outputUrl`; `mediaType?`, `spec?` | Persist a generated output and register it. `mediaType` is required only with `outputUrl`. Owner/admin mutation; voice saving may create a cloned voice.                                                                                      |

Use stored compliance when the workflow ran with Brand Guard. Otherwise use on-demand scoring
for a completed workflow, atomic, or pipeline image. Scoring is image-aware across run
families; feedback, stored compliance, and refinement still require workflow provenance.

## Generated-app management

| Tool                       | Main inputs                                                                                                     | Use and side effects                                                                                                                                                                                                                             |
| -------------------------- | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `lamina_generate_workflow` | `instruction?`; `baseAppId?`, `ops?`, `name?`, `visibility?`, `provider?`, `brandProfileId?`, `run?`, `inputs?` | Create a private app or edit a generated app in place. `ops` requires `baseAppId` and is the deterministic edit path; otherwise supply an instruction. `run: true` can dispatch immediately, so obtain approval for both creation and execution. On the planner path the response may include `editability` `{ score (0–1), subscores, notes[] }` — a deterministic measure of how tweakable the app is; a low score plus its `notes` is a cue to offer the user a refinement (via `ops`/`instruction`). |
| `lamina_app_versions`      | `appId`; `restore?`                                                                                             | Omit `restore` to list versions (read). Supplying a version restores it (write), snapshots the current graph first, and requires elevated ownership. Confirm before restore.                                                                     |
| `lamina_set_visibility`    | `appId`, `visibility`                                                                                           | Change reach to `private`, `shared`, or `public`. This is a publication mutation; confirm the exact audience.                                                                                                                                    |

After generating or editing an app, use the returned parameter keys rather than assuming its
old schema. If `run` was not approved, leave it false and call `lamina_run` only after inputs
and cost are confirmed.

## Credits and checkout

| Tool             | Main inputs | Use and boundary                                                                                                                                                                                               |
| ---------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lamina_credits` | none        | Read workspace balance and available packages.                                                                                                                                                                 |
| `lamina_topup`   | `packageId` | Create a Stripe-hosted checkout URL. This does not complete payment or prove credits were added. Give the URL to the user, never collect card data, and recheck credits after they report completing checkout. |

Keep checkout separate from creative execution. A low balance does not authorize creating a
checkout session without the user's request.

## Reliable cross-tool workflows

### App chosen manually

`lamina_discover` → `lamina_describe` → collect inputs and approval → `lamina_run` →
`lamina_status`

To chain two apps manually, wait for the first to complete, take a returned `outputs[].url`, and
pass it to the exact URL parameter key described by the second app.

### Brief routed to an app

`lamina_create` → branch on `status` → ask questions → `lamina_run` → `lamina_status`

Remember that create returns no execution run ID because it does not execute.

### Bespoke reusable app

`lamina_create` (`unmatched` with suggestion) → approval → `lamina_generate_workflow` →
collect inputs → `lamina_run` → `lamina_status` → optional approved
`lamina_set_visibility`

### Direct atomic asset

`lamina_models_list` → `lamina_models_describe` → approval →
`lamina_generate_image`, `lamina_generate_video`, or `lamina_generate_audio` → `lamina_status`

### Narrated multi-shot video

Draft narration → `lamina_compose_formats` → `lamina_compose_plan` → show cost and obtain
approval → `lamina_compose_video` → `lamina_status`

### Brand-aware workflow image

`lamina_brand` plus `lamina_brand_kit` → approved `lamina_run({ applyBrand: true, ... })` →
`lamina_status` → `lamina_brand_compliance` or supported `lamina_brand_score` → show result →
optional approved feedback or budget-bounded refinement

### Save a successful output

Complete a supported run → show the selected output → obtain owner/admin approval →
`lamina_save_to_brand_kit` → verify with `lamina_brand_kit`
