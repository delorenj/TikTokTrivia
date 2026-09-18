# RUN.md template

Written to `s3://tiktoktrivia/<run-id>/RUN.md`, updated as the run proceeds. This is where Jarad
reads what happened without interrupting her, and it is what an old run has instead of a memory.

Keep it short. Decisions and costs, not narration.

```markdown
# <run-id>

## Brief
<topic, angle, who would feel smart getting these right — in her words where possible>

## Questions
Pool: <n> candidates · Shipped: <n>
Source: <where they came from>
Verified: <how each answer was confirmed>
Cut from the set: <which, and why>
Her calls: <what she changed, kept, or rejected when asked>

## Look
Form: <the one visual form chosen>
Runner-up: <what it beat, and why it lost>
House style: <read from carries-house-style | written this run | not yet>

## Generation
Stills: fal-ai/ideogram/v3 · anchor: <image_urls from frame 1 | style_codes> · seed: <n>
        image_size: <explicit value> · rendering_speed: <value>
Narration: vox · voice: <description used> · bytes at narration/qNN.wav

## Cuts
cuts/<name>-00.mp4  depth 0  <timestamp>   <-- must exist before any depth 1
cuts/<name>-01.mp4  depth 1  <timestamp>   added: <animation | captions | sound>

## Cost
Stills: <n> images x $<rate> = $<total>
Everything else: free (vox, ffmpeg)
Total: $<total>

## Outcome
<shipped | sent back to [step] | scrapped>  <date>
<what she said, verbatim, if she said something worth keeping>
```

If a step fails, leave the prefix in place and record which step and what the error was. Never
silently retry something that costs money — say what failed, what a retry would cost, and ask.
