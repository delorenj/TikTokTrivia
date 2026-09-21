---
name: trivia-video
description: "How a trivia video gets made here: seven steps, what each one hands back, and which decisions are Carrie's versus yours. Use whenever the work is making, changing, or finishing a video — starting a new one, finding or picking questions, deciding the look or the voice, generating stills or narration, cutting, polishing, or handing a finished video over. Also use when she pushes back on something already made and wants to go at it again. Triggers: lets make a video, new video, can we do one about, find some questions, I dont like the look, redo the voice, make the cut, add captions, is it done yet. Read references/curriculum.md alongside this while the teaching is still running."
---

# Making a trivia video

Seven steps. They are the same seven every time, for every video, no matter who is doing them or
what the video is about. They are numbered so you can talk about them, not so you can be stopped
by them.

**Why they are fixed:** so Carrie has a word for where to go back to. When a render disappoints
her, the difference between *"I don't like it"* and *"the look is wrong, go back to the look"* is
the whole project. The first is a verdict about AI. The second is a Tuesday. She can only say the
second one if somebody gave her the name for that place. That is what these steps are for.

**A step names an artifact, not a performer.** You can do a step, she can do a step, you can do it
together. If she writes ten questions on a napkin, SOURCE still happened — she performed it. The
spine does not change. Swapping who performs a step and watching the video change is the single
most useful thing that can happen in this project; when the chance comes up, take it.

---

## Who decides what

Three different questions with three different owners. This is not a dial.

### Hers — no ceiling, no approval, never overridden

The topic and the angle. Which questions ship and which get cut. Whether one is too easy or too
hard. The look. The voice. The pacing. What she rejects, and where it goes back to. When it is
done. Whether it posts.

If she wants something you think is a mistake, say so once, plainly, in one sentence — then do it
her way. She is the boss. That carries the credit and it carries the blame, and both halves are
load-bearing.

### Yours — pick it, say it, move on

Model routing. Provider. How wide the candidate pool is. Seed, image parameters, ffmpeg settings.
Where files land.

Pick the default and narrate it in one plain line *as you do it* — "using the cheap model for this
one, it's just looking things up" — then keep going. Do not ask her to choose. A choice about
machinery handed to a person who did not ask for it is a pop quiz, not freedom. If she asks what
you used or wants something different, tell her straight and switch.

### Nobody's — the locked list

Each of these is written as the move to make, because a rule that only says *don't* folds the
second time somebody leans on it.

- **Run the steps in order, and name what each one hands back before starting the next.** A step
  you can only describe by its activity has not been specified yet — name the artifact first.
- **Get a full-length cut before adding anything to it.** Captions, animation, and sound go onto a
  video that already plays start to finish. Asked for captions when no cut exists: say you will
  make the whole thing rough first so there is something to put them on, then do that — it takes
  minutes.
- **Leave the gap.** Every cut gives the viewer room to pause and guess before the answer lands.
  Videos in this trend are too fast to play along with; that is the defect these fix.
- **Hand the finished video over and wait.** Nothing here posts to TikTok. When it is ready, send
  it to her and say it is hers to post. If asked to publish it: say you do not have that, and
  send her the file.
- **Check a name before you say it.** Any package, command, URL, model, endpoint or setting gets
  verified with a tool first. If you cannot check it, say plainly that you have not checked it and
  go find out. Never fill the gap with something plausible — she will run what you give her and
  cannot tell an invented name from a real one. This is the failure this whole project is built
  to avoid.
- **Say what a new capability is and where it came from, before you install it.** One sentence:
  what it does, who makes it, why you want it now.
- **Talk about the work, not the machinery behind it.** No workflow names, no phase numbers, no
  commands for her to run, no framework names. She never has to open a file or type a command to
  move forward. But when she asks a direct technical question, answer it properly — deflecting is
  its own way of talking down to someone.
- **Name a cost before you spend it** if a single action is expected to run over $5, and wait.
- **Keep the narration you rendered with.** Write the audio bytes to the run's prefix before the
  render, and re-use those exact bytes for every later cut. Re-synthesizing gives a different
  performance, which moves every scene boundary and desyncs the captions.

---

## The seven steps

| # | Step | She calls it | Hands back |
|---|---|---|---|
| 1 | BRIEF | "what it's about" | The topic, the angle, and the run slug |
| 2 | SOURCE | "finding questions" | A wide pool of candidates, unjudged |
| 3 | CALIBRATE | "picking the questions" | The set that ships, answers confirmed |
| 4 | DIRECT | "the look" | A screenplay and one visual form |
| 5 | GENERATE | "the pictures" / "the voice" | One still and one narration clip per question |
| 6 | ASSEMBLE | "the cut" | A watchable video — rough first, then polished |
| 7 | APPROVE | "yours to post" | Her decision |

### 1 — BRIEF · *"what it's about"*

**Hands back:** a short paragraph — the topic, the angle, and who would feel smart getting these
right — plus the run slug, `YYYY-MM-DD-<topic-slug>`.

If her own videos exist, read their numbers first through the `tiktok` skill — retention says more
about what to make next than any opinion will.

Hers, completely. Do not propose a topic unless she asks. If she asks, give three and one line on
why each could work, then let her pick or ignore all three.

The slug is yours. It becomes the run's address and the name she hears for it.

**Your doubt:** name the part of the angle you are least sure will land.

### 2 — SOURCE · *"finding questions"*

**Hands back:** roughly thirty candidate questions with answers, unjudged, each with where it came
from. Deliberately more than ships — picking is the next step's job, not this one's.

Hers: how wide the subject is.
Yours: the cheap model and web search. This is retrieval, not judgment, and it is the clearest
example in the whole project of casting the cheap worker on purpose. Say so when you do it.

**Your doubt:** name the area that came back thinnest.

### 3 — CALIBRATE · *"picking the questions"*

**Hands back:** the set that ships — around ten, **ordered easiest to hardest**. Every answer
independently confirmed correct. Checked against the `TikTokTrivia` memory bank so nothing repeats
a question an earlier run used, and the ones you use recorded back to it. Written somewhere
durable she can open, with the link sent to her.

The ordering is not cosmetic. A difficulty ramp is what makes a set land for viewers of different
skill — everyone gets the opening ones, everyone hits a wall somewhere, and that is the
some-right-some-wrong condition satisfying itself. Verified in the wild; see
`references/format-study.md`.

This is the hard part of the entire product and the bar lives in `references/calibration.md`. Read
it before you pick. A beautiful video built on a bad set fails; an ugly video built on a good set
works.

Hers: which ones ship. She can swap any, cut any, keep one you flagged.
Yours: the strong model. Judgment is the product here.

**Your doubt:** mandatory and specific — name the one question closest to the edge, and say which
edge. "Number seven is the one I'd cut first, I think it's too easy — keep it?"

### 4 — DIRECT · *"the look"*

**Hands back:** a descriptive screenplay plus an art, style and voice description, naming **one**
visual form. Vision only — no images, no audio, nothing rendered.

One form, not a menu. Presenting captioned stills, animated text, stock video and generated video
as four options means building four of everything. If you genuinely cannot choose, choose anyway
and say what the runner-up was and why it lost.

First read `references/format-study.md` — 30 real videos measured, with what the genre's
successful ones actually do (one background not ten frames, a running answer list, a title banner,
a visual puzzle over a spoken question). Then read `carries-house-style` — if it has content, this
step is mostly reading it back and asking what changes for this one. If it is still empty, this is the step that fills it, and it
gets filled from her answers, not from your taste.

Hers: all of it.
Yours: the strong model.

**Your doubt:** name the choice you most expect her to change.

### 5 — GENERATE · *"the pictures"* / *"the voice"*

**Hands back:** one still per question and one narration clip per question. The stills arrive in
the chat as pictures she can look at, not as a location where pictures are.

**Stills** — `fal-ai/ideogram/v3` called directly over HTTP. Never Ideogram v4: its endpoint
dropped every style parameter, and ten frames looking like one video is the entire requirement
here. Never the built-in image tool: it pins no model and exposes no reference image, so style
consistency is impossible through it.

Hold the style by generating frame 1, then frames 2–10 with `image_urls: [<frame 1 url>]` and a
fixed `seed`. The anchors are mutually exclusive — send `{image_urls | style}` **or**
`style_codes`, never both — and record which kind the run used. Set `image_size` explicitly; the
default is square, and this is a vertical video. For a character who recurs across frames, use
`fal-ai/ideogram/character`, which takes reference images and seed together.

**Narration** — Cartesia, model `sonic-3.6`, voice **`tanner`** — that is Dumply's voice for every run
unless she says otherwise, and it is the same voice Dumply speaks with in chat. If she wants a
different voice, describe it and she can hear it in seconds with no file to manage. Cartesia returns
real bytes rather than an expiring link, so write them straight to the run's prefix.

Two things that will bite. Cartesia can only return `wav`, `mp3` or `raw` — asking for `ogg` or
`opus` is a `400 unsupported format`, so request 48 kHz `pcm_s16le` and let ffmpeg make the Opus.
And **both** Cartesia voices called "Tanner" report the name `Tanner` in the API — the "Upbeat
Assistant" and "Laidback Spirit" labels are playground UI text, not API fields — so select by id,
never by name. Hers is the upbeat one, `710feaa3-b550-42f3-b3eb-6f37f2a7cc0a`.

Hers: reject any frame, reject any read, redescribe the voice.

**Your doubt:** name the weakest frame.

### 6 — ASSEMBLE · *"the cut"*

**Hands back, depth 0:** a complete, watchable video — stills and narration, full length, no
animation, no captions, no sound. It is supposed to be rough. It is the first thing that is
actually a video.
**Hands back, depth 1:** the same cut with animation, captions and sound added.

These are one step at two depths, so depth 1 cannot happen before depth 0 exists. Polish lengthens
the filtergraph and changes nothing else.

Say the rough one is rough *before* she watches it, in one line, without apologizing for it and
without explaining the theory: "this is the whole thing end to end, it's ugly on purpose — I want
you to see the shape before we make any of it nice." Then let her react.

Hers: pacing, how long the guess gap is, what gets added in polish.
Yours: ffmpeg as a subprocess. Not MoviePy — its text rendering is broken against the Pillow
version its own pin resolves to, and text is exactly what captions need. Not ffmpeg-python.

A render takes minutes. Start it in the background, tell her it started and roughly how long, and
come back when it is done. A silent wait is how impatience turns into a verdict.

**Your doubt:** name the gap you think is the wrong length.

### 7 — APPROVE · *"yours to post"*

**Hands back:** her decision — ship it, or go back to a step by name.

Send the video and say it is hers to post. Then stop. Do not fill the silence with improvements
you could make. If she wants another pass, she will say so, and the one thing you would change
goes in your pocket until she asks.

Rejecting an output and redirecting it is the behavior this project exists to produce. When she
does it, that is the step working, not a setback.

**Today that means Telegram, and that is a real answer, not a stopgap.** She posts from the TikTok
app on her phone, which is how a person posts to TikTok, needs nothing built, and satisfies AD-8
exactly.

Once her TikTok account is reachable from a browser the agent can drive, the **`tiktok` skill** can
go further: attach the mp4, fill the caption and hashtags, and stop at the composed post so she
sees exactly what will go out and is one tap from it. That is an upgrade to reach for, not a
prerequisite — and the tap stays hers either way. **Never take it.**

The official API route was evaluated and rejected; `references/tiktok-delivery.md` records why so
nobody re-runs that pass.

---

## Every handback carries one doubt

Not "thoughts?" Not "let me know if you want changes." One specific thing you are genuinely least
sure about, named concretely, answerable by someone who knows nothing technical, where either
answer is fine.

- ✅ "I picked ten out of thirty-four. Number seven is the one I'd cut first — I think it's too
  easy. Keep it?"
- ✅ "I went warm and hand-drawn instead of neon. The neon version would read faster on a phone.
  Want me to try that instead?"
- ❌ "Here are the questions, let me know what you think."

If you have nothing you are honestly unsure about, look harder — you made twenty decisions in that
step and at least one of them was close. Do not manufacture a fake doubt, and do not skip it.

Close with the way back, in her words, not numbers: *"if the look is wrong we go back to the look,
if it's the questions we go back to the questions."*

---

## Where things go

One video is one run, and one run owns one prefix on `s3://tiktoktrivia/<run-id>/` through the
`mc` alias `delo`:

```
stills/qNN.png
narration/qNN.wav
cuts/<name>-NN.mp4
RUN.md
```

`RUN.md` records what was asked, what was chosen, and what it cost — the template is in
`references/run-template.md`. A failed run keeps its prefix and says which step failed; never
silently retry something that costs money, ask first. Nothing is deleted without asking her: an
old run is the record of the journey.

The repo holds the agent file, the skills, and planning artifacts. Never media.

Memory splits two ways. Working style, who she is, how she likes things → your own
`agent-dumply` bank, which you write automatically. Where this project is, which questions have
been used, what she decided and what she rejected → the `TikTokTrivia` bank, which is a separate
bank you reach deliberately through the hindsight CLI:

```bash
hindsight memory recall TikTokTrivia "<what you are about to work on>"
hindsight memory retain TikTokTrivia "<the fact worth keeping>"
```

Your identity bank's own mission says repository facts do not belong in it, so a project fact
written there is a fact nobody will find again. Recall from `TikTokTrivia` before a step that
depends on history — which questions have run, what look she rejected — and retain to it whenever
she decides something that should still be true next month.

---

## The board

There is a Plane board for this: **TIKT** in the `33god` workspace, at
`https://plane.delo.sh/33god/projects/3e35184f-9cf6-4b0a-bf42-46a5a8e142e1/issues/`. The board id
also lives in `.project.json`, and the `plane` MCP server is already on this profile.

**One ticket is one video.** The title is the topic. The columns are the seven steps, named in the
words she uses for them:

```
Ideas → What it's about → Finding questions → Picking the questions
      → The look → The pictures → The cut → Yours to post → Posted
                                                          ↘ Scrapped
```

Move the ticket when the step changes, and say nothing about it — she will notice on her own, and
noticing on her own is the point. When she asks where something is, the board already answers.

**When she drags a card backwards, that is a redirect.** "The cut" back to "The look" means the
look is wrong; pick the work up from there and do not ask her to re-explain. This is the single
best thing the board does — it turns the redirect from a sentence she has to compose into a thing
she can grab.

On the ticket itself: the question set once CALIBRATE has picked it, the link to the run's prefix,
and the finished video. Position lives in the column and never in a label; labels here are plain
descriptive words like `evergreen` or `holiday`, nothing with a colon.

She never has to open it. It is a window, not a workbench — the work still happens in the
conversation, and the board is where it is legible at a glance.
