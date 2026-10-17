# Brand MCP

## Connection

Public setup guide: https://docs.uselamina.ai/guides/commerce-mcp.

Connect the agent host to `https://app.uselamina.ai/mcp/commerce` using remote HTTP MCP
and complete browser OAuth, selecting the intended billing workspace. This is a separate
connection from the main Lamina MCP, using the same Lamina authorization service. A website
login alone is not MCP authorization. Access also requires workspace membership and an
operator grant; public app visibility alone is insufficient.

The server sends instructions during initialization, descriptions and schemas in
`tools/list`, and the same guide at `commerce://guide`. Read that resource if the host
does not expose initialization instructions. Call `commerce_describe` for current app
availability, required fields, allowed options and person-reference previews. Do not
treat this installed reference as a replacement for live app contracts.

## Choose the app

<!-- BEGIN GENERATED BRAND CATALOG -->

| App | Preparation tool | Full output pack |
| --- | --- | --- |
| Fashion Catalogue (`fashion_catalog`) | `commerce_prepare_catalog` | Cropped Image (image); Front (image); Back (image); Side (image); Full Body (image); Catalog video (video) |
| Catalogue Photos and Video (`catalog_upgrade`) | `commerce_prepare_catalog_upgrade` | Front view (image); Side View (image); Close-up (image); Video (video) |
| Product Ad Banners (`product_banners`) | `commerce_prepare_banners` | Ad 1 (image, 9:16); Ad 2 (image, 9:16); Ad 3 (image, 9:16); Video Ad 1 (video, 5 seconds); Video Ad 2 (video, 5 seconds); Video Ad 3 (video, 5 seconds) |
| Virtual Try-On (`virtual_try_on`) | `commerce_prepare_try_on` | Selfie Output (image, 9:16); Full body Output (image, 9:16); Video (video) |
| Apparel Ad Video (`apparel_ad_video`) | `commerce_prepare_apparel_video` | Ad video (video, 20 seconds) |
| Listing Images & URL to Product Ad (`non_apparel_ad_video`) | `commerce_prepare_product_video` | Ad Video (video, 10 seconds, 9:16) |

Use optional output_ids from commerce_describe to select deliverables; omission prepares the full pack. Prerequisite generation is automatic and included in the estimate. Ask only the missing inputs returned by preparation, together. Reuse retained_inputs when supplying answers. Images-only skips downstream video work; video-only includes required images. Selection happens before generation, not just when displaying results. Internal AI models remain fixed.

The full packs above are available choices, not a requirement to generate every output. Read commerce_describe for current output IDs, availability, allowed choices and person photos. Input requirements depend on the selected outputs; preparation returns the actual missing questions. A selected producer retains its authored number of files.

Product inputs contain productName and an images array; omit unknown optional product facts. For person_reference, send either `{source: "preset", preset_id: "ID from commerce_describe"}` or `{source: "upload", asset_url: "HTTPS image URL"}`. Upload local files first. Do not copy example or saved sample photos into missing inputs.

### Fashion Catalogue

Create a styled fashion shoot from front, back and side photos of the same garment. Choose a preset person or custom person photo, garment framing, location and day/night lighting. Full output pack: Cropped Image (image); Front (image); Back (image); Side (image); Full Body (image); Catalog video (video). Returns a prepared plan before generation; no Shopify connection to Lamina is required.

| Input | Type | Requirement / default | Description |
| --- | --- | --- | --- |
| `front_image_url` | image | Required when needed by the selected outputs | Front view of the actual garment. |
| `back_image_url` | image | Required when needed by the selected outputs | Back view of the same garment. |
| `side_image_url` | image | Required when needed by the selected outputs | Side or three-quarter view. Required until the no-side-image path is verified. |
| `person_reference` | person_reference | Required when needed by the selected outputs | Use the supplied person photo, or a preset ID from commerce_describe. If the user delegates selection, choose a listed preset and show it in the review; otherwise ask. |
| `garment_type` | choice | Required when needed by the selected outputs | Upper Wear, Bottom Wear or Full Body. |
| `shoot_mode` | choice | Optional; default Day Shoot | Day or night lighting; defaults to Day Shoot. |
| `environment` | choice | Required when needed by the selected outputs | Choose a supported shoot location. |
| `instructions` | text | Optional; default blank | Optional additional creative instructions; defaults to blank. |

### Catalogue Photos and Video

Upgrade apparel listing media from a product name and photos. Choose a preset person or custom person photo; body description and location are optional. Full output pack: Front view (image); Side View (image); Close-up (image); Video (video). Returns a prepared plan before generation; no Shopify connection to Lamina is required.

| Input | Type | Requirement / default | Description |
| --- | --- | --- | --- |
| `product` | product | Required when needed by the selected outputs | Product name and image URLs from the user or connector. These are the only required product properties. Omit unknown optional properties; never invent a vendor, description or price. No store connection required. |
| `person_reference` | person_reference | Required when needed by the selected outputs | Use the supplied person photo, or a preset ID from commerce_describe. If selection is delegated, choose a listed preset and show it in the review; never invent an ID. |
| `body_description` | text | Optional; default blank | Optional body styling description; blank leaves the model reference in control. |
| `environment` | choice | Optional; default System Detected | Supported location; System Detected lets the workflow select it. |

### Product Ad Banners

Create campaign banners and animated variants using product photos, your logo, brand styling, headline, CTA and audience. Subheading and offer are optional. Choose this for banner creatives; choose a film app for a story ad. Full output pack: Ad 1 (image, 9:16); Ad 2 (image, 9:16); Ad 3 (image, 9:16); Video Ad 1 (video, 5 seconds); Video Ad 2 (video, 5 seconds); Video Ad 3 (video, 5 seconds). Returns a prepared plan before generation; no Shopify connection to Lamina is required.

| Input | Type | Requirement / default | Description |
| --- | --- | --- | --- |
| `product` | product | Required when needed by the selected outputs | Product name and image URLs from the user or connector. These are the only required product properties. Omit unknown optional properties; never invent a vendor, description or price. No store connection required. |
| `logo_image_url` | image | Required when needed by the selected outputs | Actual brand logo. |
| `brand_colors` | text | Required when needed by the selected outputs | Brand colors, supplied or explicitly delegated. |
| `brand_fonts` | text | Required when needed by the selected outputs | Brand fonts, supplied or explicitly delegated. |
| `tone` | text | Required when needed by the selected outputs | Tone of voice for the campaign. |
| `headline` | text | Required when needed by the selected outputs | Headline based on real product facts; agent proposals require review. |
| `cta` | text | Required when needed by the selected outputs | Call to action approved for this campaign. |
| `geography` | text | Required when needed by the selected outputs | Target market; never reuse the sample market. |
| `audience` | text | Required when needed by the selected outputs | Intended audience. |
| `subheading` | text | Optional; default blank | Optional subheading. |
| `offer` | text | Optional; default blank | Actual promotional offer, or blank for no offer. Do not invent discounts. |

### Virtual Try-On

Show an actual customer wearing apparel using their photo and product name/images. Use the customer photo; there is no preset-person choice. Full output pack: Selfie Output (image, 9:16); Full body Output (image, 9:16); Video (video). Returns a prepared plan before generation; no Shopify connection to Lamina is required.

| Input | Type | Requirement / default | Description |
| --- | --- | --- | --- |
| `product` | product | Required when needed by the selected outputs | Product name and image URLs from the user or connector. These are the only required product properties. Omit unknown optional properties; never invent a vendor, description or price. No store connection required. |
| `selfie_image_url` | image | Required when needed by the selected outputs | The actual customer photo; no sample or preset substitute. |

### Apparel Ad Video

Create a clothing campaign film with music, logo and timed text overlays. Supply clothing product name/images, brand logo, target geography and audience. Full output pack: Ad video (video, 20 seconds). Returns a prepared plan before generation; no Shopify connection to Lamina is required.

| Input | Type | Requirement / default | Description |
| --- | --- | --- | --- |
| `product` | product | Required when needed by the selected outputs | Product name and image URLs from the user or connector. These are the only required product properties. Omit unknown optional properties; never invent a vendor, description or price. No store connection required. |
| `logo_image_url` | image | Required when needed by the selected outputs | Actual brand logo. |
| `geography` | text | Required when needed by the selected outputs | Target geography. |
| `audience` | text | Required when needed by the selected outputs | Target audience. |

### Listing Images & URL to Product Ad

Create a product ad from a listing URL and ordered listing photos. The authored app requests four scenes, feature text and music, without voiceover. No logo is required. The URL is passed as text; webpage retrieval and product-fact accuracy are not verified. Full output pack: Ad Video (video, 10 seconds, 9:16). Returns a prepared plan before generation; no Shopify connection to Lamina is required.

| Input | Type | Requirement / default | Description |
| --- | --- | --- | --- |
| `listing_url` | listing_url | Required when needed by the selected outputs | HTTPS product listing URL supplied by the user or connector. Passed as text to the authored workflow; page retrieval is not guaranteed. |
| `listing_images` | image URL array (1–8, ordered) | Required when needed by the selected outputs | One to eight actual product listing image URLs, in the desired reference order. Upload local files first. Unused image slots are cleared; sample photos are never reused. |

Note: The authored app receives the listing URL as text; it does not have verified webpage retrieval. Review generated feature claims against the real product.

Note: The authored video prompt mentions nine references. With fewer than eight listing photos, visual fidelity needs review; no sample photos are substituted.

<!-- END GENERATED BRAND CATALOG -->

If the requested ad format is unclear, ask one format question using the available apps. Required
facts may come from the user's existing Shopify connector, files or answers. Do not
reconnect Shopify to Lamina or invent missing products, logos, people or offers. With Brand MCP contract 1.3.0 or later, use optional `output_ids` from
`commerce_describe.outputs[].id` for images only, video only or particular deliverables.
Omit it only when the full pack is intended. Required upstream generation is automatic:
a video may need images, and the estimate includes them. Do not ask for separate
prerequisite approval. Preparation returns selected outputs and only relevant missing
questions. Reuse `retained_inputs` and its selection when answering them. The base
`required_for_run` flag is qualified by `required_for_output_ids`; preparation determines
what is actually missing. Never drop invalid IDs and retry as a full run. A changed
selection needs a new plan. These IDs select existing producers, not arbitrary file
counts or variants; they are different from thumbnail `preview_output_ids`. This is
Brand-only; it does not alter general app or other MCP contracts.

## Inputs and execution

1. Read `commerce_credits` and describe the chosen app. Use exact preset IDs/choices.
   Person-reference selection means preset/custom photos, not the internal AI model.
2. For local images, call `commerce_upload_asset` per file, PUT the bytes to its signed
   URL, then pass each `assetUrl` in the right field. Upload issuance alone is not a
   completed upload. The host needs filesystem/HTTP capability; uploads are public and
   require the user's acceptance. Use approved accessible image URLs when appropriate.
3. Call the matching preparation tool with known inputs. For `needs_input`, ask the
   returned missing questions together, retaining supplied values. A description is not
   a prepared plan. Never submit model/provider/graph overrides.
4. Show the actual review, included outputs and estimate. Respect existing authorization
   for the concrete task; obtain any missing authorization before spending. Execute the
   returned `planId` and `planFingerprint` with positive `maxCredits` and a unique
   `idempotencyKey`. The estimate check is not a hard provider-spend ceiling.
5. Poll `commerce_status` with bounded intervals until completed, failed or cancelled.
   A running response can contain no outputs even at 100% step progress. On uncertain
   dispatch, retry exactly the same execute arguments/key instead of creating a new run.
6. Present the returned media from MCP, retaining named original-file links. Inspect those media files before
   describing quality; `verification: verified` checks structure/provenance, not visual
   correctness. Do not replace missing MCP results with saved canvas/database outputs.

Report the actual failure and retain the run ID. Do not blindly repeat paid runs.
If the app's authored graph changes, an operator may need to refresh its reviewed
contract before a newly prepared run is allowed. Exact per-run settled credits may be
unavailable; do not substitute a concurrent workspace balance difference.

Brand MCP does not publish to Shopify. The agent may use its own authorized Shopify
connector to publish returned files when the user requests that action.

## Present references and outputs

Preserve labelled `image` content blocks and `resource_link` blocks from describe,
prepare and status. Printing only JSON/`structuredContent` hides the previews from
humans. Use the host's actual media display API. MCP Apps hosts can mount the Brand
gallery/player; native-image hosts can show thumbnails and use their video preview
tool. Text-only hosts need named original-file links and an explicit display limitation.
Do not say “shown above” unless a visible surface was actually produced. Do not rely
only on remote Markdown image tables or download files to bypass display restrictions.

All photos and outputs appear in `presentation.items`, with labels, IDs and original
URLs. Embedded thumbnails are capped at four per call. To show later photos, pass
`app_key` and `preview_preset_ids` to describe, or `runId` and `preview_output_ids` to
status. Use IDs from that response. `inlinePreview: included` means the response has
an image block, not proof of human-visible rendering. Unavailable thumbnails do not
mean generation failed. Preserve failed/cancelled status with any partial outputs.

Show the exact selected preset/custom reference in the preparation review. Choosing
a gallery photo requests preparation, never execution. Refresh the existing run to
retrieve results without generating again. After a presentation update, reload tool
and resource discovery; existing OAuth grants and endpoint/tool names remain valid.
