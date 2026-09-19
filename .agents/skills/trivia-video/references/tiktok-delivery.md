# Getting a finished video to TikTok

Read this during APPROVE. Researched 2026-09-19 against developers.tiktok.com.

**Nothing here changes what APPROVE does today.** Today the finished mp4 goes to Carrie over
Telegram and waits, per AD-8. This file describes a better path that is *mostly* verified, names
the one thing that is not, and gives the experiment that settles it. Do not wire it up until that
experiment has run.

## The path that fits our constraint

`POST /v2/post/publish/inbox/video/init/` with scope **`video.upload`** uploads a video into the
user's TikTok **inbox as a draft**. She gets a notification, taps it, TikTok's own editor opens,
and she posts from there. No code path can publish — the API has no way to.

This is a stronger AD-8 than Telegram delivery, not a weaker one. The gate stays absolute, she
stops having to download a file and re-upload it by hand, and the status endpoint reports
`SEND_TO_USER_INBOX` when it lands and `PUBLISH_COMPLETE` once she posts — which is a CAP-6
measurement we currently have no way to take at all.

The other endpoint, **Direct Post** (`video.publish` scope), publishes straight to her profile.
**We do not use it and should never add that scope.** Not having the capability is better than
having it and promising not to call it.

### The flow

1. `GET https://www.tiktok.com/v2/auth/authorize/?client_key=…&scope=video.upload&response_type=code&redirect_uri=…&state=…`
2. `POST https://open.tiktokapis.com/v2/oauth/token/` (`grant_type=authorization_code`)
   → `access_token` (24h) and `refresh_token` (365d). **The refresh token rotates** — the response
   may carry a different one and you must store the new one or you lose access.
3. `POST https://open.tiktokapis.com/v2/post/publish/inbox/video/init/` with
   `{"source_info":{"source":"FILE_UPLOAD","video_size":N,"chunk_size":N,"total_chunk_count":1}}`
   → `publish_id` and an `upload_url` **valid for one hour**.
4. `PUT {upload_url}` with `Content-Range: bytes 0-{N-1}/{N}` and `Content-Type: video/mp4` → `201`.
5. Poll `POST https://open.tiktokapis.com/v2/post/publish/status/fetch/` with the `publish_id`.

Use `FILE_UPLOAD`, not `PULL_FROM_URL`. Pull-from-url needs domain or URL-prefix ownership
verified in the developer portal, an https URL with no redirects that stays live for the whole
download window. File-upload needs none of that — push the bytes from the run prefix. (The
Content Sharing Guidelines state a style preference the other way, but that guidance is aimed at
audited public apps.)

The one-hour `upload_url` expiry is the same shape of trap as vox's 3600s links in AD-5: get the
bytes moving promptly and do not hold a URL across a long gap.

### Constraints — our render already satisfies every one

| | Required | Ours |
|---|---|---|
| Container / codec | MP4 (recommended), WebM, MOV / H.264 (recommended), H.265, VP8, VP9 | MP4 / H.264 ✅ |
| Framerate | 23–60 FPS | 30 ✅ |
| Dimensions | 360–4096 px both axes | 1080×1920 ✅ |
| Duration | ≤ 10 minutes | ~2 min ✅ |
| File size | ≤ 4 GB | ✅ |
| Chunking | under 5 MB must be a single chunk; chunks 5–64 MB, final up to 128 MB, max 1000, sequential | single chunk ✅ |

Rate limits: **6 init requests/min** per user token, **30 status polls/min**, and at most
**5 pending shares per 24 hours** — exceeding it returns `spam_risk_too_many_pending_share`. That
last one is a real production ceiling: five staged-but-unposted videos and staging stops until she
clears some. If a backlog ever builds, that is the limit that bites.

## Production app review is a dead end — do not attempt it

TikTok's Content Sharing Guidelines name this exact project shape as unacceptable:

> Not acceptable: A utility tool to help upload contents to the account(s) you or your team
> manages. ❌

Reinforced by the App Review FAQ: *"Beta or development versions, incomplete apps, and test
versions are not encouraged and will not be approved for integration with TikTok in most cases."*

So **Sandbox is the destination, not a stepping stone.** It needs no app review and supports up to
10 target accounts. We need one.

## Getting a sandbox

An ordinary TikTok for Developers account, created from the signup page with an email. No business
account and no organization required (the docs call joining an organization "highly recommended
but not required"), and no app-store listing — that requirement attaches to app review, which
sandbox skips.

1. Manage apps → **Connect an app**.
2. On the app page, switch the toggle next to the app's name to **Sandbox**.
3. **Create Sandbox**, name it.
4. Edit App details and add products: **Login Kit** and **Content Posting API**.
5. **Apply changes** — the configuration is inert until you do.

Limits: 5 sandboxes per app, 10 target user accounts.

**Carrie has to be present for one step.** To add an account as a target user you must supply its
login credentials — the portal redirects to a TikTok login and whoever holds the account logs in
and accepts the TikTok Developer Terms of Service. This does not breach CAP-8, which governs the
work of making videos, not one-time setup. But it cannot be done for her, so it is a five-minute
thing to do sitting next to her, once, and it should be framed as such rather than sprung on her.

## The one thing that is NOT verified

**No page on developers.tiktok.com states that the inbox endpoint works from a sandbox app.**

The only sandbox-scoped statement about this product is an exclusion — *"Sandbox mode does not
offer access to Content Posting API for public videos or Data Portability API"* — and *"for public
videos"* is load-bearing phrasing that TikTok never defines. Two readings both survive:

- **Favorable:** sandbox blocks only the public-visibility path (Direct Post / `video.publish`).
  The inbox flow never makes anything public, so it works. Supported by a note in the
  app-registration doc that *"For Sandbox environments, URL verification is only required for
  Content Posting API"* — implying the product exists in sandbox at all.
- **Unfavorable:** "for public videos" is clumsy phrasing for the whole product and sandbox blocks
  all of it.

The docs do not choose between these and neither will we. **Settle it by experiment, not by
reasoning.** This is exactly the case AD-9 exists for.

### The experiment — about 30 minutes, no code, no app review

1. Create the developer account, connect an app, toggle to Sandbox, create a sandbox.
2. Add Content Posting API and the `video.upload` scope.
   **Gate 1:** if `video.upload` cannot be added in sandbox, you have the answer in ten minutes.
3. Add a **throwaway TikTok account you control** as the target user — not Carrie's.
4. Run the Login Kit flow for `scope=video.upload`, get an `access_token`.
5. `curl` the init endpoint with a ~2 MB MP4, single chunk.
   **Gate 2:** `200` with `publish_id` + `upload_url` means it works. `403`,
   `scope_not_authorized`, or product-unavailable means it does not.
6. If 200: `PUT` the bytes, poll for `SEND_TO_USER_INBOX`, then check the throwaway account's
   inbox on a phone.

Step 6 settles a second open question for free: post that draft from the app and see whether it
lands public or is forced private. That tells us whether the unaudited-app "private viewing mode"
restriction reaches this path — which matters, because a video Carrie cannot make public is not a
delivery mechanism.

**Until both gates pass, APPROVE keeps doing what it does now: send her the mp4 over Telegram and
wait.**
