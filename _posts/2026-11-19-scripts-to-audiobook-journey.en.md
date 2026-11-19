---
layout: post
title: "I Stopped Generating Dialogues: Design Tradeoffs in Scripts to Audiobook"
date: 2026-11-19 00:00:00 +0800
tags: [language-learning, tts, audio, open-source, design-tradeoffs]
excerpt: "What language practice actually lacks is not pronunciation correction but dialogues worth practising. This build turns existing scripts into multi-character audio, with as little model dependency as possible."
lang: en
slug: scripts-to-audiobook-journey
permalink: /blog/scripts-to-audiobook-journey
---

![Scripts to Audiobook: script input and character setup](/images/scripts-to-audiobook-dark.png)

**The short version:** this tool does not generate dialogues — it turns good ones into practice-ready multi-character audio. Every trade-off points the same way: it should run on demand, not generate on demand.


## It started with a practice session that didn't work

With an LLM, the biggest problem in speaking practice is usually not pronunciation — it is a dialogue that was never worth practising.

I used Doubao and ChatGPT for speaking practice. They produce a dialogue instantly, which is convenient — but two lines in, the problem is obvious, and it isn't pronunciation. It's the dialogue itself: the logic jumps, the depth is thin, and there's no goal. The worst outcome in speaking practice is finishing a session that felt smooth while the material was never worth practising.

So I changed direction. Instead of asking a model to invent a conversation, go find conversations that are already good: a scene from a show you liked, a paragraph you wrote, a short exchange someone shared.

## Material doesn't have to be a textbook

Practice material does not have to be a textbook: a short exchange, a fragment, even something a model generated and you filtered will do.

One condition matters more than format: the material can be a short exchange, a fragment, even something a model generated and I filtered. The only requirement is that **it passed my judgement — something I like and would willingly practise again**.

Swapping "generate" for "select" changes the product. The tool is no longer a dialogue generator; it is a pipeline that turns good material into practice-ready audio.

## Choosing TTS: all four providers have free tiers

Without paid APIs, the option space is wider than it looks.

The first real constraint was voice synthesis: avoid paid APIs where possible, but don't let quality collapse.

Four providers ended up integrated: Edge TTS, ElevenLabs, iFlytek, Baidu. They differ in quota and in how you call them. iFlytek's interface is less conventional than the rest, which cost extra integration time; the other three follow shapes you have seen before.

Voices are handled differently per provider. Edge TTS keeps **all** of its English and Chinese voices, since it is free and plentiful. The other three keep only the voices I tested and found usable — fewer, but each one works.

## Multiple characters means voices must differ

The fastest way this kind of audio fails is that the listener cannot tell who is speaking.

An easy detail to miss: a dialogue isn't read by one person. It has roles.

Once roles exist, voice choice stops being arbitrary. If two characters sound alike, the listening experience collapses — you can't tell who is speaking. So beyond language, accent, gender, age and narrator suitability, the matcher reserves a separate term: a **diversity penalty** that lowers the chance of reusing a voice already assigned.

Those weights are not academic. They were tuned by ear. The test was simple: does this sound like two people talking?

## MP3 + SRT, no video

Language practice does not need visuals, so the output is just MP3 and SRT.

I hesitated over output format. Video looks more complete, but it eats attention: resolution, caption styling, footage editing — each one a project of its own.

The core here is language practice, pronunciation above all. **Visuals don't matter.** So the output is two things: a merged MP3 and a timed SRT. Smaller files, faster loading, ready to practise against.

## A deliberate constraint: depend on models as little as possible

Use AI to produce reusable code, but ship something that runs without any model — that was a deliberate constraint.

Several projects from that period followed the same rule. The reasoning is simple: depending on a model means depending on a network, a quota, and someone else's interface staying stable. A tool like this needs to *run on demand*, not *generate on demand*.

That is why it looks like an engineering artifact: four TTS providers behind one interface, a static JSON voice catalog, and matching logic computed locally.

Would I add AI back if I rebuilt it? I considered it. The answer: **yes, but only as scaffolding.** Helping a user plug in resources they already have, or bring their own paid quota into the loop — that adaptation work differs per provider, and a model is well suited to it. But if that scaffolding ever becomes one uniform capability, I would rather use AI to generate the scaffolding than have it sit in the runtime path.

## This version only solved half of it

This build only completes the first two steps — script to multi-character audio; recognition and scoring arrived in another project.

The original need had four steps: find a good script → turn it into multi-character audio → capture my reading → analyse my pronunciation.

This build completes the first two. Speech recognition and pronunciation scoring came later, in a separate project — a harder chain: real-time, low latency, and scores that can be explained.

Together, the two projects are what I originally set out to build.

## Links

- Repository: [hugcosmos/scripts-to-audiobook](https://github.com/hugcosmos/scripts-to-audiobook)
- The other half: [SceneLine](https://github.com/hugcosmos/sceneline)

<!--
Image plan (to add later; 1600px wide, 16:10, consistent light theme):
1. Landing view with script input — reuse existing /images/scripts-to-audiobook-dark.png
2. Character voice cards — proves differentiated matching across roles
3. Output (MP3 + SRT) — proves the deliverable shape
-->