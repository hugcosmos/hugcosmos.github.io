---
layout: post
title: "Whole Books to Audiobooks: Four Constraints Behind Book to Audiobook"
date: 2026-11-19 00:00:00 +0800
tags: [audiobooks, tts, local-models, open-source, design-tradeoffs]
excerpt: "When the input grows from a dialogue to an entire book, quota, speed, chapters and container all force design changes. These are the four constraints behind Book to Audiobook."
lang: en
slug: convert-whole-books-to-audiobooks
permalink: /blog/en/convert-whole-books-to-audiobooks
---

**The short version:** almost every technical decision in this project was forced by one thing — the input got long. Quota runs out, local models run slow, chapters have to exist, and the container has to hold a whole book. It complements Scripts to Audiobook: one handles a few minutes of dialogue, the other handles hundreds of thousands of words.

## I just wanted to listen while lying down

The original motivation was concrete: staring at a screen to read an ebook hurts my eyes. No matter how sharp the text is, two hours of reading is tiring. Audio does not need my eyes — press play and it runs.

That sounds like an odd thing for a product person to say, but it shaped everything downstream: I never designed for looking, only for the continuity of listening.

## Constraint one: quota

In Scripts to Audiobook, a dialogue lasts minutes. Cloud TTS is fine there — every provider has a free tier, and short content leaves plenty of headroom.

A whole book is different. Hundreds of thousands of words means hours of audio, and the same free tier gets consumed in one run, or simply cannot cover it. So the default path here became **local models**: Kokoro on Intel Macs (82M parameters, CPU inference), Qwen3 MLX on Apple Silicon.

The cost is real — models must be downloaded, they occupy hundreds of megabytes, and Kokoro is pinned below Python 3.12. What it buys is no per-character billing, which is decisive for a book.

## Constraint two: speed

Local models are slow. Even with multithreading, a full book takes time.

I decided that does not matter, because **consumption happens after generation**. Drop a book in before bed, listen the next morning; the machine waits, not me. What would be unacceptable is waiting a minute for one line of dialogue — that is the short-content case, not this one.

So there is no real-time design here at all. It behaves like a batch job.

## Constraint three: chapters

The chapters are not mine to invent — the book already has them. Splitting follows the natural structure: whatever the table of contents says, that is the chapter list.

The real difficulty is format. EPUB, MOBI and AZW3 are relatively structured; PDF is not. A PDF is the product of layout, not structure, so extraction comes with noise. For those files I allow manual edits: list chapters, view one, edit one.

One detail I insisted on: **edits are stored separately, and the original text is never overwritten.** An edited chapter carries an `[edited]` tag, and conversion prefers the edited copy. You can undo a bad edit; the source stays intact.

## Constraint four: container

Why M4B by default? A practical reason: I develop on macOS, and M4B is the native format that carries chapter metadata — players show chapters and remember position.

But there is not just one output. A single chapter can be exported as MP3, for when you want to try one first; the whole book is finally merged into a single M4B with chapter markers, so it can also be consumed straight through. **One body of content, two ways to consume it.**

## Free providers: instant start, with the risk stated

Besides local models, the project also integrates Edge, ElevenLabs, iFlytek and Baidu. They stay because they are free and work instantly — no model download, just a key.

Edge TTS carries a fact that belongs in the README: it is not an official Microsoft API, and sustained batch use may violate their terms. I put that sentence next to six alternatives instead of hiding it.

That is a trade-off. For personal and research use it is convenient enough; for commercial use, switch to a local model or an official API. Write it down and the user decides.

## The versions were pushed by bugs

From v0.1.9 to v0.2.0, six releases in a month. What drove them was not a feature list but bugs and details I had not thought of at the start — a PDF that split into fragmentary chapters, a conversion that had to resume after interruption.

Iteration like this never shows up in release highlights, but it is why the tool actually became usable.

## If this became a commercial product, I would take the choice away

Today the project asks you to choose a provider and tune a voice. Commercially, I would do the opposite: **replace the providers entirely.**

Not to save money — to guarantee consistency. A commercial product needs control over every voice detail before it can promise "it will sound like this." The next step is to analyse the uploaded document, decide whether it reads as narration or dialogue, and pick the matching provider and voice automatically — the user never needs to understand TTS, only to receive a good version.

Letting users choose pushes complexity onto them. Choosing on their behalf means owning the result.

## Links

- Repository: [hugcosmos/book-to-audiobook](https://github.com/hugcosmos/book-to-audiobook)
- The complementary project: [Scripts to Audiobook](https://github.com/hugcosmos/scripts-to-audiobook)

<!--
Image plan (to add later; 1600px wide, 16:10, consistent light theme):
1. CLI chapter listing — proves splitting follows the book's structure
2. An edited chapter carrying [edited] — proves the original-text protection
3. Output directory with M4B and single-chapter MP3 — proves two consumption modes
-->