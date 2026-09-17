# Question calibration

The product's quality bar. Render quality is not it — a beautiful video built on a bad question set
fails, and an ugly video built on a good one works. This is also the part the source calls hard and
explicitly not a solved one-shot prompt.

## The band

A set is calibrated when a viewer gets **some right and some wrong in the same set**.

- Too easy is boring — nothing to feel smart about.
- Too hard is ridiculous and kills the format outright.
- The target feeling is excitement at knowing the answer, which requires that knowing it was not
  guaranteed.

## The ritual it serves

The format works because the viewer pauses and guesses before the answer lands. That is the actual
viewing behavior this is calibrated for, and it has two consequences:

1. **The cut must leave a gap** between question and answer. Existing videos in this trend are too
   fast to play along with — that is a defect in the source material and a thing these videos can fix
   rather than inherit.
2. **Captions must be legible** at the speed the gap allows, since reading is how the viewer holds
   the question during the pause.

## Where the sweet spot lives

The one selection strategy the source offers: **niche areas people believe they are smart about
because they know that area.** A question inside someone's self-identified domain is the one they are
excited to get right, and it is the mechanism by which a set can be simultaneously hard and
satisfying rather than hard and alienating.

This is a heuristic, not a procedure. It has not been tested and it is the only guidance available
for the problem named as the hard part.

## Shape of the step

Not "produce ten questions." The step needs a **candidate-then-select** shape: generate a wider pool,
then apply the calibration criterion to choose the set that ships. Judging happens against the band
above, not against whether the questions are interesting.

## Working sizing

Ten questions filling roughly two minutes, read aloud with answer gaps. Treated as the output
contract; the source offers it as an example figure, so it is open.

## The test

Play a finished set with the pause-and-guess ritual.

- **Pass** — some right, some wrong, within the same set.
- **Fail** — all right (too easy) or all wrong (ridiculous, no fun).

## Known gaps

- **No verification.** Nothing in the pipeline checks that an answer is correct, and question-finding
  is deliberately assigned to the cheapest model. A wrong answer in a trivia video is a product
  failure, not a cosmetic one.
- **No dedupe.** The step is designed to run daily or weekly. Nothing described so far keeps a run
  from resurfacing questions an earlier run already used.
- **No provenance rule.** Whether questions are generated, drawn from quiz databases, or lifted from
  other creators' videos is unsettled, and the three differ in whether the set can be reused.
