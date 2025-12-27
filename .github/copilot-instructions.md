# Copilot instructions (pandoc-videofilter)

## What this project is
- A Pandoc JSON filter that turns image URLs like `vimeo:95369722` / `youtube:FEFETKhhq8w` into embedded HTML `<iframe>`s.
- Only the HTML output format is transformed; other formats should remain unchanged.

## Architecture (where to look)
- Entry point: [app/Main.hs](../app/Main.hs) calls `Lib.process`.
- Filter core: [src/Lib.hs](../src/Lib.hs)
  - `process` wires Pandoc `toJSONFilter` + reads env vars.
  - `inlineHtmlVideo` matches `Image` inlines and returns `RawInline (Format "html") ...`.
  - Dimensions are extracted from the image *alt text* (e.g. alt text `400x300` on a `youtube:...` image) via `Text.Pandoc.Walk.query`.
- Parsing: [src/VideoParser.hs](../src/VideoParser.hs)
  - `parseVideoId` accepts only `vimeo:<digits>` and `youtube:<id>`.
  - `parseDimensions` expects `WIDTHxHEIGHT` (e.g. `400x300`).
- Rendering: [src/VideoRenderer.hs](../src/VideoRenderer.hs)
  - `renderVideoEmbed` uses `lucid` to build the `<iframe>` and includes fullscreen attrs.
  - Dimension attrs are optional and come from `Maybe Dimensions`.
- Optional wrapper: [src/WrapperElement.hs](../src/WrapperElement.hs)
  - Wraps the iframe in `<div class="...">...</div>` when configured.

## Runtime configuration (env vars)
- `VIDEO_DIMENSIONS`: default `WIDTHxHEIGHT` used when alt text does not specify size.
- `VIDEO_WRAPPER_CSS_CLASS`: if set, wraps embeds in `<div class="...">`.

## Developer workflows (commands that match this repo)
- Build (Stack): `stack build`
- Test: `stack test`
- Produce a local binary:
  - `stack build --copy-bins --local-bin-path=./bin/` (matches CI in [circle.yml](../circle.yml))
  - This repo’s Stack config also sets `local-bin-path: binaries` in [stack.yaml](../stack.yaml).
- Lint (as CI does): `git ls-files | grep '\.l\?hs$' | xargs stack exec -- hlint -X QuasiQuotes "$@"`

## Project-specific conventions to preserve
- Keep HTML output stable: tests compare the full rendered HTML string (including entity escaping).
  - See [test/Spec.hs](../test/Spec.hs) for exact expected output.
- When adding a new video service, update all three places consistently:
  - [src/Definitions.hs](../src/Definitions.hs) (`VideoService`)
  - [src/VideoParser.hs](../src/VideoParser.hs) (`videoParser`)
  - [src/VideoRenderer.hs](../src/VideoRenderer.hs) (`videoUrl`)
  - Then extend [test/Spec.hs](../test/Spec.hs) with exact-string expectations.
