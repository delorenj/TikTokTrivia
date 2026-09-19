---
name: tiktok
description: "How to actually reach TikTok: search and study what performs, read a creator's own analytics including watch retention, and put a finished video in front of them to post. TikTok's official APIs cannot do these things for a project like this, so the interface is a real logged-in browser driven through ego-browser. Use whenever the work touches TikTok itself — researching a format, checking how a posted video did, or getting a rendered video ready to publish. Also use when someone needs setting up with browser access for the first time. Triggers: look at tiktok, what is doing well on tiktok, research the format, how did it do, check the numbers, post it, upload it, put it on tiktok, its ready to go up."
---

# Reaching TikTok

**The browser is the interface. There is no usable API here.** That is a settled conclusion, not a
gap waiting to be filled — `../trivia-video/references/tiktok-delivery.md` records exactly which
official products were evaluated and why each is closed to this project. Do not go back around
that loop.

What a real logged-in browser gives you, all verified working on 2026-09-19:

| Job | Needs whose account? |
| --- | --- |
| Search and study what performs | **Anyone's.** Any logged-in session can search. |
| Read watch retention and traffic sources | Hers — it is her own analytics |
| Put a finished video in front of her to post | Hers |

## How you drive it

`ego-browser` runs a real Chromium on a Mac carrying genuine Chrome logins, bridged over ssh. It
is on the PATH of the machine you run on. Every call is a heredoc of Node executed on the page:

```bash
ego-browser nodejs <<'EOF'
const task = await useOrCreateTaskSpace('trivia-research');
await gotoAndWait('https://www.tiktok.com/search?q=trivia%20quiz');
await new Promise(r => setTimeout(r, 8000));
cliLog(await js(`document.querySelectorAll('a[href*="/video/"]').length`));
EOF
```

Things that will bite you if you do not know them:

- **`cliLog` is the only way to get output back.** A value you merely compute is lost.
- **No state survives between heredocs.** Re-attach every round with `useOrCreateTaskSpace(name)`.
- **TikTok renders late.** `gotoAndWait` returns well before the page is ready — wait 6–8 seconds
  before reading, or you will get an empty snapshot and conclude the page is blank. `snapshotText`
  is especially prone to this; `js('document.body.innerText')` is more reliable.
- **Close what you open.** `completeTaskSpace(name, { keep: false })` when the job is done. This is
  someone's personal laptop; do not park a logged-in session for convenience.
- **Run `ego-browser doctor`** if anything fails. It names the broken link. The usual answer is
  that the Mac is asleep. Never quietly fall back to curl or a headless browser — they cannot see
  the session and will report a login wall as an empty page.

## Job 1 — studying what performs

Any session will do; this has nothing to do with whose account is logged in. Search, collect video
URLs, then read each video's own embedded data for exact numbers.

```bash
# search -> urls
await gotoAndWait('https://www.tiktok.com/search?q=' + encodeURIComponent(query));
await new Promise(r => setTimeout(r, 8000));
const urls = await js(`JSON.stringify([...new Set([...document.querySelectorAll('a[href*="/video/"]')].map(a=>a.href))])`);
```

```bash
# one video -> exact stats, straight from the page's own payload
const out = await js(`(() => {
  const el = document.getElementById('__UNIVERSAL_DATA_FOR_REHYDRATION__');
  if (!el) return 'NO_REHYDRATION_NODE';
  const it = JSON.parse(el.textContent).__DEFAULT_SCOPE__['webapp.video-detail'].itemInfo.itemStruct;
  return JSON.stringify({author: it.author.uniqueId, dur: it.video.duration, stats: it.stats, desc: it.desc});
})()`);
```

`stats` carries `playCount`, `diggCount`, `commentCount`, `shareCount`, `collectCount`. Verified
against a second tool — the numbers are the real ones, not estimates.

**Comment rate is the metric that matters**, not likes: what people comment on a trivia video is
their score, so it measures playing along. Median in the genre is about 0.11%; good is 0.45%+.
`../trivia-video/references/format-study.md` is the standing baseline — read it before running a
new study, because the answer may already be in it.

Wait between page loads and keep runs small. This is someone's real account browsing, so behave
like it: a handful of videos on a question worth asking, not a crawl.

## Job 2 — how her own videos did

TikTok Studio, her session. `https://www.tiktok.com/tiktokstudio/analytics`.

This is the richest signal in the entire project and **no API offers it.** Overview, Content,
Viewers and Followers tabs; per-video watch retention and average watch time; traffic sources
broken down by Search / For You / Following / profile / sound; the actual search queries that
found her; and a **Download data** export.

Retention is the one that decides things. A trivia video lives or dies on whether people stay for
the answer, and this is the only place that is visible. When a video underperforms, look here
before changing anything — a drop at question three is a different problem from a drop at the
intro, and they have opposite fixes.

Feed what you learn into the next BRIEF, and retain it to the `TikTokTrivia` bank so the next run
starts from it.

## Job 3 — putting a finished video up

**Never click Post.** Not once, not to be helpful, not because it is obviously ready.

Drive `https://www.tiktok.com/tiktokstudio/upload`. There is a real `input[type=file]` accepting
`video/*`. Attach the rendered mp4, fill in the caption and hashtags, set whatever options were
agreed — then **stop at the composed post and hand it to her**, with a snapshot of exactly what
will go out and a plain sentence saying what is left to click.

This is not a limitation to work around. It is AD-8, and this path honours it better than the
alternative: she sees the actual finished post as it will appear, rather than a draft in an inbox.
Getting her all the way to a one-tap decision is the whole job; taking the tap is not yours.

If she says "just post it," the answer is that you have set it up and it is one tap, and you would
rather she took that tap. Then leave it set up.

## Setting someone up for the first time

She needs a browser carrying her TikTok login that the agent can reach. That is a one-time setup,
and it is the one place this project asks her to install something.

That does not break the rule that she never *needs* to run a command to move a phase forward,
because posting was always going to be hers — the setup serves the one step that is hers by
design. Frame it that way and it reads as handing her the keys rather than homework.

Walk her through it conversationally, one step at a time, waiting after each. Never send a list.
Tell her plainly what it does before she installs anything: it is a browser that stays logged into
her accounts so the agent can work in them, it runs on her machine, and she can watch it.

When she is set up, **verify before believing it**: load TikTok Studio and confirm the upload page
renders. A login that half-worked looks exactly like one that worked until the moment it matters.

Worth knowing: the bridge currently points at one specific Mac. If she works somewhere else, that
is a real question to settle at setup time rather than discover mid-session — check
`ego-browser doctor` names the machine you expect.

## The gate, in one line

Search freely. Read freely. Fill everything. **She posts.**
