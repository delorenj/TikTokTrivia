---
review: versions-and-verification
target: ../ARCHITECTURE-SPINE.md
reviewer: adversarial-verification pass
date: 2026-09-17
host: big-chungus
verdict: pass-with-fixes
---

# Review — Versions & External Verification

**Lens.** Every externally-named thing in the spine, independently confirmed against the live
machine and the live web. Not "is this a good design" — the paradigm (agent + skills, no
application tier) is a decision and is not under review. Only: **is every name in here true, and is
it still true today.**

**Method.** Read-only probes only. No paid image or audio generation. Local ffmpeg chains were
re-executed because they cost nothing. Every claim below carries the command or URL that produced
it.

**Headline.** The spine is unusually well-sourced — the Ideogram v4 trap, the MoviePy exclusion,
the ffmpeg build flags, vox, S3 and the MCP list all survive scrutiny, several of them exactly. But
AD-9 ("every external name in it was probed live, not recalled") is not true of two of the spine's
own statements, and one of them sits inside AD-4, the rule that guards the project's single hard
media requirement. A spine whose thesis is anti-fabrication has to be right about this, so the bar
here is unforgiving on purpose.

---

## Verification ledger

| Claim in spine | Verdict | Evidence |
| --- | --- | --- |
| ffmpeg 7.1.1 with libass, libx264, libfreetype, libfontconfig | **TRUE** | `ffmpeg -version` → `7.1.1-1ubuntu4.2`; configure line contains all four. `apt-cache policy` shows it is also the archive candidate. |
| Skeleton + polish chains run on this machine | **TRUE — re-executed** | Both re-run here, exit 0. See "ffmpeg re-execution" below. |
| MoviePy excluded: TextClip broken vs. its own resolved Pillow pin | **TRUE** | Released 2.2.1 (2025-05-21, latest) pins `pillow<12.0,>=9.2.0` → resolves **11.3.0**. MoviePy `CHANGELOG.md` carries "Fix TextClip broken with Pillow > 11.2" under **[Unreleased]** — the fix is on master, in no release. |
| ffmpeg-python excluded: no release in seven years | **TRUE** | PyPI: 0.2.0, uploaded 2019-07-06. That is 7y 2m as of today. |
| Ideogram v4 dropped every style parameter | **TRUE — exactly** | `developer.ideogram.ai/openapi.json`: `/v1/ideogram-v4/generate` accepts only `text_prompt, json_prompt, resolution, rendering_speed, enable_copyright_detection`. `/v1/ideogram-v3/generate` still carries `style_reference_images, style_codes, style_type, character_reference_images, seed`. The spine's list is verbatim correct. |
| `fal-ai/ideogram/v3` exists | **TRUE** | fal queue OpenAPI returns a schema; `category: text-to-image`. |
| AD-4's parameter names and combination | **FALSE** | See **F1**. |
| vox live at `vox.delo.sh`, backed by VoxCPM2 | **TRUE** | `/healthz` → `{"status":"ok","engines":[voxcpm ready, vibevoice not ready, elevenlabs ready]}`. MCP `initialize` → `serverInfo {name: vox, version: 3.2.4}`, instructions: "backed by VoxCPM2 with ElevenLabs fallback". HF `openbmb/VoxCPM2`: apache-2.0, 2,290,004,544 params, tagged voice-cloning + voice-design. |
| `/synthesize-url` links expire after 3600s | **TRUE** | `compose.yml: VOX_AUDIO_TTL_SECONDS: "3600"`; `app/cache.py` default 3600; sweep every 300s. |
| Artifacts to `s3.delo.sh` (MinIO) | **TRUE — credentials work** | `mc ls` with the DeLoDrive (MinIO) vault creds lists 12 buckets. But see **F6**. |
| MCP servers are vox, plane, codegraph, pjangler | **TRUE** | Rendered `config.yaml` `mcp_servers` has exactly those four. All four have cached schemas (each has connected). All binaries exist; `codegraph` resolves on the systemd user PATH. |
| `web.backend` / `search_backend` / `extract_backend` all empty | **TRUE as a config fact** | Confirmed empty strings. But the conclusion drawn from it is wrong — see **F3**. |
| `.project.json` = `TIKT`, empty `board_id` | **TRUE** | Verified. `plane.delo.sh` → 200. |
| Runs as the `dumply` profile on big-chungus | **TRUE** | `hermes-dumply-gateway.service` active + enabled since 2026-09-17 04:52:51, `WorkingDirectory=/home/delorenj/code/TikTokTrivia`. |
| Release `0408fec7…` | **TRUE** | Install dir name and `HERMES_RUNTIME_RELEASE` in the systemd drop-in both match. |
| hermes-agent **0.20.5** | **FALSE** | See **F2**. |
| "82 fleet skills" | **Off by two** | 80 `SKILL.md` under the profile. |

---

## Findings

### F1 — CRITICAL — AD-4 prescribes a parameter that does not exist and a combination the API forbids

AD-4 reads:

> Generate through fal (`fal-ai/ideogram/v3`), holding style with `style_reference_images` +
> `style_codes` + a fixed `seed`.

Against the live fal schema
(`https://fal.ai/api/openapi/queue/openapi.json?endpoint_id=fal-ai/ideogram/v3`), two things are
wrong:

1. **`style_reference_images` is not a field on this endpoint.** The style-reference field on fal is
   `image_urls` — *"A set of images to use as style references (maximum total size 10MB across all
   style references)."* The full input property list is: `image_urls, rendering_speed, color_palette,
   style_codes, style, expand_prompt, num_images, seed, sync_mode, style_preset, prompt, image_size,
   negative_prompt`. The name `style_reference_images` belongs to **Ideogram's own direct API**
   (`/v1/ideogram-v3/generate`), which the spine explicitly decided against using. AD-4 mixes the
   vocabulary of two different providers.

2. **`style_codes` cannot be combined with style references or `style`.** From the schema, verbatim:
   *"A list of 8 character hexadecimal codes representing the style of the image. **Cannot be used in
   conjunction with style_reference_images or style**."* And on `style`: *"The style type to generate
   with. **Cannot be used with style_codes**."* So the prescribed triple is not merely misnamed, it
   is mutually exclusive. Sent as written it is a 422, or silently one lever instead of two.

This is the rule that carries the hard requirement — *ten frames that look like one video* — and it
is the rule AD-9 points at when it says every external name was probed live. It was not.

**Fix.** Replace the mechanism clause in AD-4 with one of these two, both of which stay on fal and
stay on Ideogram v3:

- **Minimal (recommended).** Generate frame 1 from the art direction. Then generate frames 2–10 with
  `image_urls: [<frame 1 url>]` and a fixed `seed`, and **do not send `style_codes` or `style`.**
  This also happens to be a clean lesson beat: *we made one, then told it to match.*
- **Stronger, if a recurring character or mascot appears.** Use `fal-ai/ideogram/character`, which is
  the one endpoint in the family that accepts style references **and** character references at the
  same time: `image_urls` (style refs), `reference_image_urls` (character refs, 1 image), plus
  `seed`, `style_codes`, `style`, `negative_prompt`, `image_size`. Verified live; same fal key, no new
  account. Note fal's own docs flag that character-reference generations bill on a separate tier.

Either way, state in AD-4 that the mutually-exclusive set is `{image_urls ∪ style} XOR style_codes`,
so the next person does not rediscover it in a 422.

---

### F2 — HIGH — The stack table's hermes-agent version is wrong; the runtime is 0.20.1

The spine's Stack table says `hermes-agent | 0.20.5 (release 0408fec7)`. The release hash is right.
The version is not.

The binary the gateway actually executes reports itself:

```
$ /home/delorenj/.local/share/hermes-agent/releases/0408fec7…/.venv/bin/hermes version
Hermes Agent v0.20.1 (2026.8.13) · upstream dbf5e7a0
Install directory: …/releases/0408fec7a153e6c32c064acd2b8053917f1525f1
Up to date
```

Corroborating: that release tree's own `pyproject.toml` says `version = "0.20.1"`, and its
`site-packages` carries `hermes_agent-0.20.1.dist-info`.

Where 0.20.5 came from — two places, neither of them the running runtime:

- `~/.hermes/.update_check` contains `{"ver": "0.20.5", "behind": -1, "rev": null}`. `behind: -1` and
  `rev: null` mean that check did not resolve; `ver` there is not the installed version.
- The source checkout at `~/.hermes/hermes-agent` has `version = "0.20.5"` committed at HEAD
  (`6852bdba`) — but that branch is **2622 commits ahead of `delorenj/main` and unpushed**, and the
  deployed release is built from `dbf5e7a0` on the remote line, where pyproject still reads 0.20.1.

Note also that `hermes version` prints "Up to date" while the update check is broken. The runtime's
own self-report is not a trustworthy source here, which is worth knowing given AD-9.

**Fix.** Change the Stack row to `hermes-agent | 0.20.1 (2026.8.13), release 0408fec7`, and record
the command that produced it (`<release>/.venv/bin/hermes version`) rather than the file that did
not. If the intent was actually to run 0.20.5, that is a deploy task, not a documentation edit.

---

### F3 — HIGH — The "blocks Carrie's first session" web-search blocker is very likely a phantom, and its stated workaround is dead

The Deferred section says:

> `web.backend`, `web.search_backend` and `web.extract_backend` are all empty on this profile. …
> A `browser` block is configured and `x_search` points at `grok-4.20-reasoning`, so the capability
> may be reachable another way — but plain search is not wired. **Blocks Carrie's first session.**

The config fact is correct. The inference is not, and the hedge is worse than unverified — it is
false.

**a) Empty string is read as unset, not as disabled.** In the deployed runtime,
`agent/web_search_registry.py::_read_config_key` gates on `if isinstance(cur, str) and cur.strip()`
— an empty string returns `None`. `get_active_search_provider()` then calls `_resolve(None,
capability="search")`, which the module docstring describes as *"the path that fires when no config
key is set — pick the highest-priority backend the user actually has credentials for,"* walking
`firecrawl → parallel → tavily → exa → searxng → brave-free → ddgs`, filtered by `is_available()`.

**b) The profile already holds credentials for two of those.** The rendered `config.yaml` injects
`PARALLEL_API_KEY` and `EXA_API_KEY` from 1Password, and both references resolve (40 and 36 chars
respectively; values not printed). `FIRECRAWL_API_KEY` is present but empty, so the walk should land
on **parallel**. Nine providers ship in this release: `brave_free, ddgs, exa, firecrawl, parallel,
searxng, tavily, xai`.

**c) The stated fallback cannot work.** `x_search: grok-4.20-reasoning` is a real model — xAI
released it 2026-03-10, 1M context, $1.25/$2.50 per Mtok — but `plugins/web/xai/provider.py` requires
`XAI_API_KEY` or an xAI OAuth token in the auth store, and **there is no XAI key anywhere in this
profile's secrets block or `~/.hermes/.env`**. Writing "may be reachable another way" about a path
with no credential is precisely the move AD-9 forbids, appearing in the spine's own text.

**Fix.** Do not deprecate the deferral — verify it, then rewrite it. One command settles it: send
Dumply a question that forces a search and see which provider answers, or read the gateway log for
the resolved provider. If parallel answers, the entry becomes a one-line convention ("pin
`web.search_backend: parallel` so the backend is explicit rather than inferred"), not a blocker, and
the first lesson is unblocked today. Drop the `x_search` sentence entirely or replace it with
"unavailable — no xAI credential on this profile."

---

### F4 — MEDIUM — `image_gen.provider: fal` is configured with no model pinned, so the built-in image tool defaults to FLUX, not Ideogram

The profile has `image_gen: {provider: fal, use_gateway: false}`. In the deployed runtime,
`tools/image_generation_tool.py` sets `DEFAULT_MODEL = "fal-ai/flux-2/klein/9b"`. So if a skill says
"generate an image" and the agent reaches for its built-in tool, it gets FLUX 2 Klein 9B — not
Ideogram v3 — and AD-4's consistency guarantee silently does not apply.

Worse, pinning the built-in tool to Ideogram does not rescue it. The in-tree catalog entry is:

```python
"fal-ai/ideogram/v3": {
    "defaults": {"rendering_speed": "BALANCED", "expand_prompt": True, "style": "AUTO"},
    "supports": {"prompt", "image_size", "rendering_speed", "expand_prompt", "style", "seed"},
}
```

No `image_urls`. No `style_codes`. And its default `style: "AUTO"` is itself exclusive with
`style_codes` per F1. Through the built-in tool the only consistency lever available is `seed`,
which for Ideogram is far weaker than a style reference.

The spine's diagram does say `Shell: ffmpeg · fal HTTP`, so raw HTTP is the intended path — but
nothing in the spine says the built-in tool is off-limits, and the profile is configured in a way
that invites it.

**Fix.** Add one clause to AD-4: *stills are generated by calling `fal.run` directly, not through
the agent's built-in image tool, because that tool cannot pass style references.* Optionally set
`image_gen.provider: ""` on the profile so the wrong path is not available at all.

---

### F5 — MEDIUM — Within fal and within Ideogram, a better-fitting endpoint exists than base v3

Not a challenge to fal (explicit preference) or to the Ideogram family. Purely: among the endpoints
fal exposes, base `fal-ai/ideogram/v3` is not the one with the most consistency levers.

Verified input surfaces:

| endpoint | style refs | character refs | seed |
| --- | --- | --- | --- |
| `fal-ai/ideogram/v3` | `image_urls` (excl. with `style_codes`) | — | yes |
| `fal-ai/ideogram/character` | `image_urls` | `reference_image_urls` | yes |
| `fal-ai/flux-2-pro` | — (text-to-image only) | — | yes |
| `fal-ai/nano-banana-pro` | — (text-to-image only) | — | yes |

Worth noting because the two models most likely to be reached for as "newer and better"
(`flux-2-pro`, `nano-banana-pro`) accept **no reference image at all** on their text-to-image
endpoints, so neither beats Ideogram here. The upgrade, if one is wanted, is sideways within
Ideogram: `fal-ai/ideogram/character`. Leave base v3 as the default and name `character` in AD-4 as
the escalation when a recurring subject appears.

Pricing, confirmed current on the fal model page today: *"Your request will cost $0.03 with TURBO,
$0.06 with BALANCED, and $0.09 with QUALITY."* The memlog's figures are exactly right. Worth
carrying into the spine, since the default is BALANCED — ten stills is $0.60 a pass, and iteration
means several passes.

---

### F6 — MEDIUM — AD-7 commits artifacts to S3 but never names a bucket, and none exists for this project

AD-7 says artifacts "go to `s3.delo.sh`". The Naming convention says the run slug
`YYYY-MM-DD-<topic-slug>` "is the S3 prefix". A prefix is not an address without a bucket, and the
spine never names one.

Live bucket list on `s3.delo.sh` (credentials verified working):

```
affinity-files/  artifacts/  avacadabra/  delodocs-attachments/  excalidraw/
field-audio/     flowise/    hot/         james-brennan/         jim-brennan/
recordings/      tonnybox/
```

There is no TikTokTrivia bucket. The first render will either fail or land somewhere arbitrary and
un-findable — which defeats AD-7's own stated purpose of artifacts surviving a profile rebuild.

**Fix.** Name it in AD-7: either a dedicated `tiktoktrivia` bucket, or
`artifacts/tiktoktrivia/<run-slug>/`. Then create it. One line, and it closes the only gap between
AD-7 and a working write.

---

### F7 — LOW — Assorted precision nits

- **"82 fleet skills"** — 80 `SKILL.md` files under the profile. (The fleet-shared pool is 210, so
  the number is not that either.) Harmless in a Deferred note; noted only because a spine that
  forbids unverified numbers should not carry one.
- **AD-4's "Never Ideogram v4" guards a door fal does not have.** `fal-ai/ideogram/v4` returns
  **404** — fal exposes no v4 endpoint. The warning is factually correct about v4's parameters and
  worth keeping, but the live risk it names is only reachable by switching to Ideogram-direct. The
  trap that will actually bite is F1, which the same rule gets wrong.
- **AD-5 names an HTTP route, the profile uses MCP.** AD-5 says `/synthesize-url`; vox is wired as an
  MCP server, where the tool is `speak_url` (alongside `speak`, `list_voices_tool`). The 3600s TTL
  and the download-before-render rule hold on both paths — only the name is off. Worth fixing since
  this is a name a skill will be written against.
- **The `Fal` vault item stores `API Key` as type `STRING`, not `CONCEALED`.** Still true today
  (verified). Practical consequence for this project specifically: AD-3 invites Carrie to ask
  technical questions and the answers go to Telegram, so an `op item get` in a transcript prints the
  key in the clear in a chat she can screenshot and forward. Flipping the field type costs nothing.

---

## ffmpeg re-execution (AD-6, confirmed by running it)

Both chains were re-run on this machine rather than trusted from the memlog.

**Skeleton** — concat of stills with per-scene durations, scale/pad to portrait, narration muxed:

```
ffmpeg -f concat -safe 0 -i list.txt -i narr.m4a \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,\
pad=1080:1920:(ow-iw)/2:(oh-ih)/2,fps=30,format=yuv420p" \
  -c:v libx264 -c:a aac -b:a 128k -shortest skeleton.mp4
→ exit 0 ; ffprobe: h264, 1080, 1920, yuv420p, 30/1 + aac
```

**Polish** — Ken Burns plus burned-in ASS captions, on top of the finished skeleton:

```
ffmpeg -i skeleton.mp4 \
  -vf "zoompan=z='min(zoom+0.0008,1.12)':d=180:\
x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':s=1080x1920:fps=30,ass=cap.ass,format=yuv420p" \
  -c:v libx264 -crf 20 -preset veryfast -c:a copy polish.mp4
→ exit 0 ; ffprobe: h264, 1080, 1920, yuv420p + aac
```

AD-6 holds, including its claim that polish only lengthens the filtergraph. The `ass` filter
resolved a system font through fontconfig without any font path being supplied, which is the
libass + libfontconfig pair doing exactly what AD-6 assumes. This is the best-evidenced decision in
the document.

---

## Verdict

**pass-with-fixes.**

The research behind this spine is real and mostly excellent. The Ideogram v4 parameter list is
verbatim correct against the vendor's own OpenAPI. The MoviePy exclusion is sharper than the spine
states — the fix exists only on an unreleased master while the released pin resolves straight into
the broken range. ffmpeg, vox, VoxCPM2, S3, the MCP list, the board's absence and the deployment
host all check out. Nothing in here is invented wholesale.

What fails is narrower and more uncomfortable: the two places the spine reasons *past* what it
verified. AD-4 verified Ideogram's direct API and then wrote those parameter names into a rule that
calls fal. The Deferred entry verified that three config keys are empty and then concluded the
capability is missing, without reading the code that consumes them — and reached for an `x_search`
fallback that has no credential. Both are the same error AD-9 exists to prevent, one level up:
checking a fact, then asserting a consequence that was never checked.

Fix F1 before any skill is written against it, correct F2's version, and settle F3 with one live
query. F4–F6 are each a sentence. The paradigm is sound and out of scope; the spine is close, and
the specific corrections are all small.
