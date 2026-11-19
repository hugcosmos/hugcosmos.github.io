---
layout: post
title: "Latency First, Scoring Second: The Trade-offs Behind SceneLine"
date: 2026-11-19 00:00:00 +0800
tags: [asr, tts, speech, language-learning, open-source, design-tradeoffs]
excerpt: "In dubbing practice, what breaks first is continuity, not the score. So this project put real-time recognition first and scoring second — including admitting the scoring is not done."
lang: en
slug: sceneline-dubbing-practice-platform
permalink: /blog/sceneline-dubbing-practice-platform
---

**The short version:** this project has one hard metric — latency. If speaking gets interrupted, practice stops, so real-time recognition comes first; scoring comes second, and it is still not finished. I once scored pronunciation with a phoneme-level acoustic model, and it turned out less accurate than plain ASR text comparison.

## Completing the other half

Scripts to Audiobook only did "turn a script into multi-character audio." The original idea had four steps: find material → generate speech → capture the learner's reading → analyse pronunciation.

SceneLine covers the last two: dub along with a film scene while the system listens, recognises and scores.

## Why latency comes first

Dubbing is not a reading contest; it behaves like conversation. Once there is more than half a second of silence, attention drifts — you start wondering whether it heard you, whether you should repeat yourself.

Every technical choice here serves one goal: shrink the gap between "I finished speaking" and "it responded."

The method is to run FunASR as a resident process instead of reloading the model per request. The repository records roughly a tenfold performance gain. The price is equally direct: resident memory, slower startup, a heavier deployment. I consider it worth paying — latency is something users feel immediately; memory is not.

## Scoring: the more "professional" approach was less accurate

Pronunciation assessment usually means a phoneme-level acoustic model: slice the audio into phonemes and compare each against a reference. It sounds far more rigorous than "recognise text, then compare."

I took that road. It did not work well. The problem is that **the model introduces its own uncertainty**: it can flag a broadly correct pronunciation as wrong, or miss a real problem. The user sees the score drop without knowing why, which is worse than having no score at all.

So the current direction is reversed: **fall back to plain ASR**. Get the recognised text, align it with the source line, then decide which words were missing or misread. It is crude, but explainable — the user sees "this word was dropped" instead of an unexplained 62.

That work is unfinished. Scoring is the part I am least satisfied with, and the part that needs long-term investment.

## Don't make users memorise VoiceIDs

The technically convenient design is to let users enter a voice ID — every TTS provider has its own naming, and if you type it correctly, it works.

Now picture the real case: you upload a Friends scene with six characters. Under that design you memorise six IDs; even if you play only one role, you memorise the other five.

That is not a technical problem — it is **pushing system complexity onto the user**. So the project filters by gender and locale instead: 40-odd voices narrow to a dozen, preview one, pick it. When several roles are present, the system guarantees they get different voices.

The more mature a product is, the fewer choices the user should make. In Scripts to Audiobook that shows up as automatic voice matching; here it shows up as "you never type an ID."

## What is still unsolved

If this project could keep only one feature, I would keep real-time recognition. It decides whether the product gets used at all; scoring decides whether it is useful once used.

And scoring is currently in an unglamorous state: the direction is clear (fall back to ASR, make the alignment explainable), the work is not finished. I put it in writing because a portfolio that only shows finished states teaches the reader nothing.

A note on which parts were actually thought through: latency, voices and scoring I reworked repeatedly; the three history views, script deduplication and Docker setup were implemented by an AI in passing, looked at once, and kept. Both kinds coexisting in one project is the honest picture.

## Links

- Repository: [hugcosmos/SceneLine](https://github.com/hugcosmos/SceneLine)
- The other half: [Scripts to Audiobook](https://github.com/hugcosmos/scripts-to-audiobook)

<!--
Image plan (to add later; 1600px wide, 16:10, consistent light theme):
1. Scene selection with role assignment — proves multi-character practice
2. Live recognition in progress — proves low-latency feedback
3. The three history views — proves the statistics angle
-->