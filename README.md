# Field Notes: Directing AI to Extract Knowledge from Unstructured Video

Working notes from building an AI-assisted pipeline that converted a library of trading education videos into a structured, verified strategy specification.

I did not hand-write this system. I scoped it, directed Claude Code to build it, and audited what came out. The most useful thing I learned had nothing to do with the pipeline working, and everything to do with catching it when it was confidently wrong.

Source material is a paid subscription library and is not reproduced here. Nothing in this repository contains strategy rules, parameters, transcripts, or derived notes. What follows is method and judgment only.

## The problem

A trading methodology existed only as hours of unstructured video. It was not searchable, not comparable across sessions, and impossible to build anything on top of. Watching a video to answer one question meant scrubbing through it.

The goal was a structured specification: every rule traced to its source, stated once, in a form that could be reasoned about.

## What I built

Six stages, chained:

1. **Download.** Pull source videos.
2. **Scene detection.** Split each video at visual change points, so charts and slides become discrete segments.
3. **Transcription.** Whisper over the audio.
4. **Frame and transcript pairing.** Align each visual segment with what was said over it. This is the step that makes the rest work. A rule stated verbally is ambiguous without the chart it refers to.
5. **AI analysis.** Extract candidate rules from each paired segment.
6. **Synthesis.** Merge candidates across all videos into one specification.

## What broke

**Encoding crashes.** Fixed and moved on. Not interesting.

**Coverage gaps.** Scene detection missed segments in videos where the screen barely changed. The threshold floored at a fixed value and the gap-fill only triggered after a long interval, so slow chart adjustments on static screen recordings read as no change at all. I found this because two thin videos produced almost no segments, and had to recover them manually. Still on the fix list before any new source material gets processed.

**Optimizing a metric instead of the goal.** At one point the pipeline was tuned to produce more segments. More segments is not the objective; usable coverage is. Chasing the number produced fragmentation without adding information.

**Transcription quality.** The base Whisper model produced errors that survived into extracted rules. Upgraded to large-v3 and reprocessed. Worth noting: the errors were not obvious. They were plausible words in plausible places, which is exactly the kind of error that gets through review.

## The part that mattered

An automated verification pass reviewed the specification against the source material and reported it clean.

I did not believe it.

Not because I had evidence against it, but because I knew the domain and something felt off. Reading through by hand, I found two errors:

- **A hypothetical promoted into a rule.** The source used a specific number as an illustrative example while explaining a concept. The extraction captured it as a hard parameter. The transcription was correct. The interpretation was not.
- **A chart readout misread as a rule.** A leftover measurement on a gold futures chart, a number showing where the cursor happened to be pointing rather than anything being taught, was extracted as a 10% stop distance. Again the transcription was accurate. The value simply was not a rule.

Two errors in a specification reported as clean meant the review process was wrong, not just the output. So I ran a full three-tier audit: every rule traced back to its source, hand-verified, and tagged by whether it was concrete enough to implement, partially specified, or fundamentally discretionary.

That audit corrected roughly thirteen more rules.

**What changed in how I work:** a correctly transcribed value can still be incorrectly used. Accuracy and meaning are different checks, and automated verification is good at the first one and blind to the second. So before any number gets treated as a parameter, it now gets classified first: is this a rule, an example, or a readout? Classification comes before implementation.

That habit came out of this project and I apply it everywhere now, including AI output I had no hand in producing.

## What I would do differently

**Classify before extracting, not after.** The value-type check was built as a repair. It should have been a stage in the pipeline. Every extracted number should carry a label from the moment it is pulled.

**Verify against the thin cases first.** The coverage bug surfaced in the two sparsest videos. Those should have been the test set from the start, not the last thing checked. Systems fail at their edges and I tested the middle.

**Distrust clean reports specifically.** A verification pass returning no findings is a weaker signal than one returning findings. I got lucky that domain knowledge made me suspicious. That is not a process.

## My role, stated plainly

I directed this build. I did not write the code by hand, and I am not a developer.

What I contributed: scoping the problem, designing the pipeline stages, deciding what "done" meant, catching errors the automated review missed, and rebuilding the verification approach after it failed.

The domain knowledge is what made the audit possible. An AI model could not have caught those two errors, and neither could a developer without a trading background. That intersection is the actual skill on display here.
