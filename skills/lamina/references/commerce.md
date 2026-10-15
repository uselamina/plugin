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

| User's intent | Preparation tool | Output pack |
| --- | --- | --- |
| Styled shoot using front/back/side garment photos | `commerce_prepare_catalog` | Five images and catalogue video |
| Upgrade a product listing | `commerce_prepare_catalog_upgrade` | Three images and video |
| Branded banners | `commerce_prepare_banners` | Three images and three short animations |
| Show an actual customer wearing a garment | `commerce_prepare_try_on` | Two images and try-on video |
| Clothing campaign film | `commerce_prepare_apparel_video` | Final ad film |
| Other-product campaign film | `commerce_prepare_product_video` | Base and text-overlay ad films |

An ambiguous “ad” needs one format question: banner pack or campaign film. Required
facts may come from the user's existing Shopify connector, files or answers. Do not
reconnect Shopify to Lamina or invent missing products, logos, people or offers. Each
tool prepares its complete output pack, including video; there is no image-only switch.

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
