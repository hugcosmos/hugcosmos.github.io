---
layout: post
title: "Three Tools That Don't Ask for an Account: The Price of Local-First"
date: 2026-11-19 00:00:00 +0800
tags: [tools, chrome-extension, local-first, open-source, design-tradeoffs]
excerpt: "What these three tools share is not a stack but a rule: no sign-up, no upload, no server. Here is what local-first saves and what it costs."
lang: en
slug: local-first-tools
permalink: /blog/en/local-first-tools
---

**The short version:** three tools — a LAN book server for old Kindles, an extension that masks burned-in subtitles, and an extension that reads pages aloud with Edge voices — share one rule rather than one stack: **no account required**. No email, no upload, no backend. What that saves is privacy and friction; what it costs is distribution and installation.

## The common part: no account system

Most tools open with a login. These three do not. You open them, use them, close them, and the data stays on your machine.

That is a trade-off, not a principle: the user of a small tool is usually just one person, so an account system means maintaining something nobody asked for, and turning a five-minute job into a registration flow.

## An old Kindle doesn't need a cable

The first tool solves something concrete: old Kindles have no convenient sync. Plug in, find the folder, drag files — the process takes longer than the reading.

The approach is deliberately crude. A small Flask server runs on the computer, scans the book directory, and renders a list. The Kindle opens `http://<lan-ip>:<port>` in its own browser and downloads mobi, epub, azw3, pdf or txt with one tap.

The details came later: chunked transfer so large files don't drop, automatic LAN IP detection so you don't look it up, and a single-file script so `pip install flask` is the whole setup.

No cloud, no account, no sync conflicts — because **it never needs to know which book you are reading**.

## Masking subtitles so you stop reading them

The second tool is smaller still: a Chrome extension that covers burned-in subtitles.

It started with listening practice. Platform subtitles cannot be turned off, and your eyes drift to them, so listening practice quietly becomes reading practice. The extension lays a mask over the region so only your ears can do the work.

Its constraints are strict: Manifest V3, zero permissions, zero dependencies, all code local, working on Bilibili, YouTube, Netflix, TED and Coursera.

Zero permissions is not purity — it is credibility. An extension that can read every page you visit is a harder sell than one that only paints a layer when you ask it to. **Whether people dare install it** decides whether this kind of tool exists at all.

## Reading pages aloud without the built-in voice

The third tool is ReadAloud: Edge neural voices reading pages, with word-by-word highlighting.

Browsers ship the Web Speech API, which makes sound in a single line of code. I didn't use it: few voices, uneven quality, and poor control over rate and phrasing.

So the path is harder: `edge-tts` invoked through native messaging. Audio plays in an offscreen document (an MV3 service worker cannot play media directly), and highlighting follows `audio.currentTime` on a 60 ms timer, marking one word at a time.

The cost is installation: run a script to install dependencies, register the native host, reload the extension. That step turns some people away; what it buys is voice quality and control.

The genuinely hard part is irregular pages. WeRead draws text onto a canvas, so the DOM contains nothing — the only option is injecting a script that intercepts `CanvasRenderingContext2D.fillText` and captures every character's coordinates and font size, then overlays highlighting. ChineseAll is Swiper plus PDF.js: read the active slide's text layer and handle auto page-turning. QQ Reading, Tianya and Douban each need their own extractor.

None of this generalises. You write one site at a time.

## The three costs of local-first

Making a tool unobtrusive means accepting three things:

1. **Distribution runs through GitHub.** No app-store review means no storefront; people find you by search and word of mouth. Release tags and clear install steps become mandatory work.
2. **Platform binding gets tighter.** Native messaging and canvas interception hug a specific browser and specific sites. When they change, you change.
3. **Maintenance stays with you.** There is no server to push upgrades; every user's old copy is their own.

What those costs buy is something plain: **you don't have to trust anyone**. Not that my server ignores your reading list, not that the extension avoids your passwords — the code sits in a folder on your machine.

## What belongs in a toolbox

None of these became products, and I don't intend them to. The test is simple: **how often does one person hit this problem?**

Changing books a few times a year, watching a handful of hard-subtitled videos, reading long articles — the needs are real, but the frequency can't support a subscription and doesn't justify running a server. They belong in a toolbox, free to take.

The inverse also holds: if you use a small tool every day, it may deserve to be a product — by then you are no longer its user but its maintainer.

## Links

- [micro-apps](https://github.com/hugcosmos/micro-apps): kindle-sync and the universal subtitle mask
- [edge-read-chrome](https://github.com/hugcosmos/edge-read-chrome): ReadAloud

<!--
Image plan (to add later):
1. Book list on the Kindle's own browser (a phone photo of the screen is fine)
2. Subtitles masked vs unmasked, same frame, side by side
3. ReadAloud word highlighting in action
-->