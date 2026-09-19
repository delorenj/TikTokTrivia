# What actually works — a study of 30 real trivia videos

Read this during DIRECT, and read the calibration section during CALIBRATE. Everything here was
measured, not recalled.

## Method, so you can age it out and redo it

Sampled **2026-09-19** from `tiktok.com/tag/triviaquiz`, top 30 videos surfaced on the tag page.
For each: read the page's own `__UNIVERSAL_DATA_FOR_REHYDRATION__` payload for exact duration,
dimensions and engagement counts, then captured timed screenshot sequences of four videos to see
composition and the question cycle.

Two caveats that bound every number below. **One tag, one day** — this is what `#triviaquiz`
surfaced, not a random sample of the genre, and the tag page is personalized and ranked. And
**play counts are lifetime while the videos differ in age**, so rates (like %, comment %) are more
comparable than absolute plays. Treat everything as directional.

**Before redoing this, read the standing below — it may not be the right thing to do again.**

Tooling notes. `yt-dlp` 2026.08.19 fails on TikTok out of the box (`Unexpected response from
webpage request`), but it is **not** broken — the cause is yt-dlp issue #17604, TikTok blocking
its `chrome-150` impersonation target, and overriding the user agent fixes it:

```bash
yt-dlp --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36 OPR/118.0.0.0" <url>
```

Verified today; it returned the same duration and play count as the browser method, which
cross-validates the numbers below. Cookies were never the problem. `gallery-dl` 1.32.13 also works
with no flags at all. Direct CDN fetches of `playAddr` still 403 without session cookies.

### Standing: refresh it through the browser, and keep runs small

This is a baseline, not a subscription. When it ages out, refresh it deliberately through the
`tiktok` skill — a real logged-in session searching TikTok the way a person does, which is the
supported route and needs nobody's particular account. Read this file first; the answer to a new
question is often already here.

Decision on record (Jarad, 2026-09-19): studying public trivia videos from a logged-in browser is
in scope for this project. The considered position is that `robots.txt` names crawlers and this is
a person's session, and that the scale involved — a handful of videos to answer a real question —
is indistinguishable from browsing. Keep it that way: ask a question worth asking, pull what
answers it, stop. Do not put it on a schedule and do not turn it into a crawl.

## The sample

30 videos, 1.4k to 16.6M plays. Median duration **124s**; mean 155s; range 62–523s. Half are
under two minutes, and only 4 of 30 run past four minutes.

Median like-rate **3.9%**. Median comment-rate **0.11%**.

## Finding 1 — our two-minute, ten-question contract is right

Where a video states its question count, seconds-per-question lands between **8.1 and 13.4, median
10.4**. Our working contract of ten questions in roughly two minutes is 12s per question — inside
the observed band, at the generous end. Median duration in the wild is 124s. **Keep the contract.**

Question counts cluster at two sizes: **6–7** (short, 62–81s) and **15–25** (long, 130–250s).
Ten is between the two clusters. Nothing says ten is wrong, but be aware it is not a standard size;
the genre's own attractors are "can you get 6/6" and "all 20."

## Finding 2 — the gap is about one second, and that is deliberate

This corrects an assumption in our spec. Seconds-per-question is **not** the answer gap — it is the
whole cycle: question appears, gap, answer, transition. Measured on `@funwithtrivia`, the video's
own persistent subtitle reads **"History Trivia · One Second to Answer."** The short gap is
advertised as the product, not an accident of sloppy editing.

So when the spec says existing videos are "too fast to play along with," that is a **deliberate
deviation from a working convention**, not a defect being fixed. It may well be the right
deviation — it is our differentiator — but go in knowing the genre chose speed on purpose and
these creators are getting 3–8% like rates with it. If a longer gap is the bet, it should be
visible enough that a viewer notices they were given time.

## Finding 3 — the running answer list is the genre's signature

Present in every text-based format sampled, across creators who otherwise share nothing: a
**numbered list down the left, 1) to N), that fills in as answers are revealed** and stays on
screen the whole video. `@overnightquiz1` (animated text on a still background) and
`@funwithtrivia` (a man talking to a webcam) both do it.

It does three jobs at once: it shows progress, it lets a viewer keep score without pausing, and it
gives them something to comment. **Adopt it.** It is the cheapest device in the study and it is
the one thing everything successful shares.

Also common: a persistent title banner at the top stating the challenge ("GUESS THE 20 PERIODIC
TABLE ELEMENTS BY THEIR MISSING LETTERS"), a progress bar for the whole video, and a hard
white-flash cut between questions.

## Finding 4 — difficulty ramps within the video, easy first

Verified on `@overnightquiz1`'s periodic-table video by watching the answer list fill: **Carbon,
Silver, Gold, Oxygen, Copper** — unmistakably easiest first. Thumbnails of two sibling videos by
the same creator show an explicit `EASY / MEDIUM / HARD / EXPERT` ladder printed down the left
next to the numbers. (The ladder labels are read from thumbnails, not verified in-video —
lower confidence than the ordering, which is certain.)

This is a better mechanism than the flat band in `calibration.md`, and it should change how
CALIBRATE picks. A band asks "is this set hard enough for the average viewer," which has no good
answer because viewers differ. **A ramp asks nothing** — it guarantees every viewer gets some
right at the start and hits their wall somewhere, which is exactly the "some right, some wrong"
pass condition, self-calibrating across skill levels. Order the shipped set easiest to hardest and
the band takes care of itself.

## Finding 5 — the best play-along signal is comments, not likes

Comment-rate is the metric that tracks people actually playing, because what they comment is their
score. Median in the sample is 0.11%. Four videos beat it by 4–6x, at **0.45%–0.64%** — and all
four are the same creator running the same format:

| Video | Plays | Like % | Comment % |
|---|---|---|---|
| `@overnightquiz1` 20 human body parts | 718k | 7.2% | **0.64%** |
| `@overnightquiz1` 20 physics terms | 310k | 8.7% | **0.58%** |
| `@overnightquiz1` 20 periodic elements | 192k | 6.4% | **0.45%** |
| `@overnightquiz1` 20 country/world words | 17k | 4.3% | **0.45%** |

Their like-rates are also top-of-sample. Note the confound honestly: this is **one creator, one
format** — it may be the format, or it may be them. But two things about that format are worth
copying regardless.

**The mechanic is visual, not verbal.** The puzzle is a word with letters missing — `S_LV_R`,
`O_YG_N`, `Z_NC`. You solve it by *looking*, so one second is genuinely enough, and it works with
sound off. Read-don't-listen is probably why this format survives the one-second gap that would
kill a spoken question.

**Every topic is a niche domain** — the periodic table, physics terms, human body parts, farm
animals, electricity words. Not "general knowledge." This is direct evidence for the one
calibration strategy our spec already had on faith: niche areas people believe they are smart
about. It is the highest-engagement cluster in the study. Stop treating it as an untested
heuristic and start treating it as the default.

## Finding 6 — one background per video, not one image per question

The format with the best engagement uses **a single static background image for the entire video**
— a blue stone in one, a blue flower in another — with all the content as text overlaid on top.
Not ten generated frames. Not one image per question.

This matters for GENERATE and for AD-4. The whole reason AD-4 exists is holding style consistent
across ten frames, which is hard and costs $0.30–0.90 a run. **The proven-cheap baseline does not
have that problem because it does not have ten frames.** One background is trivially consistent
with itself and costs one image.

This is not an instruction to abandon per-question stills — distinctive visuals are a legitimate
place to be better than the genre, and Carrie may want exactly that. It is a warning not to
*assume* ten stills are required. The cheap version is what is winning. If we spend on ten frames,
spend it as a deliberate bet, and consider testing one background against ten frames on two
otherwise-identical videos.

## Finding 7 — the biggest video in the sample is one we cannot make

The top video, `@extramediummedia` at 16.6M plays, is a **street game show**: real contestants
outdoors, a physical buzzer, an on-screen scoreboard, a rebus card. Our spec already rules this
out — a stills-and-voice pipeline cannot produce it and staging it would depict interviews that
never happened. That call still looks right.

The consequence is just worth stating plainly: the realistic ceiling for the format we *can* make
is the next tier down, around **13M for a well-made short text quiz** and 0.5–5M for a good one.
Not 16M. That is a fine ceiling. Nobody should measure video one against the street show.

## The three formats, and which are ours

| Format | Example | Can we make it? |
|---|---|---|
| Text/puzzle over one static background | `@overnightquiz1` | **Yes** — and it is the best engagement in the sample |
| Talking head + picture-in-picture inserts | `@funwithtrivia` | No — needs a person on camera |
| Street game show with real contestants | `@extramediummedia` | No — ruled out by the spec, correctly |

## What to carry into a brief

- Target **100–140 seconds**. Median in the wild is 124.
- **Ten to twenty questions**, ordered **easiest to hardest**.
- A **niche domain** the viewer thinks they know, not general knowledge.
- A **running numbered answer list** on screen the whole time.
- A **persistent title banner** stating the challenge and the count.
- **One background**, consistent for the whole video, with the content as text on top.
- Prefer a **visual puzzle** over a spoken question if the format allows — it survives a short gap
  and it works with the sound off.
- Ask for the score in the description. Every high-comment video does.

## What this study does not know

- Retention and watch-through **for these creators' videos** — not visible from outside. Likes and
  comments are the proxies here. Note this limit does *not* apply to Carrie's own videos: TikTok
  Studio shows her retention curves, average watch time and traffic sources directly, and that is
  far better data than anything in this file. See the `tiktok` skill.
- Whether a longer answer gap helps or hurts. Nothing in the sample tries it, which is the gap our
  videos are meant to fill — so this is untested territory, and our own first videos are the
  experiment.
- Whether `@overnightquiz1`'s numbers come from the format or the creator. One creator, four videos.
- Anything about audio. Screenshots are silent, so nothing here describes music, voice or sound
  design.
