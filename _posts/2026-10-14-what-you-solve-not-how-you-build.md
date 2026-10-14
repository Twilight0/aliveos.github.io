---
permalink: /news/what-you-solve-not-how-you-build/
layout: post
title: "What You Solve Beats How You Build: Notes on Software in the Age of AI"
date: 2026-10-14 10:00:00 +0000
author: The AliveOS Project
---


Something fundamental changed in software development over the last couple of years, and most discussions about it miss the point entirely. People argue about which model writes better code, whether juniors are obsolete, whether the craft is dead. All of that is noise. The real shift is this: **the "how" has been commoditized, so the only thing left that matters is the "what" — which problem you chose to solve.**

Let me explain what I mean, because it changes how you should work — and how quietly.

## The "how" is now cheap

For most of my career, implementation was the moat. Knowing how to parse hostile HTML at 2 AM, how to keep a Kodi addon alive across three Python migrations, how to debug a segfault in a thumbnail loader — that knowledge was scarce, hard-won, and therefore valuable.

Then AI-assisted development happened, and I will say it plainly because the numbers are mine: my productivity went up roughly tenfold. The boring 80% — boilerplate, test matrices, rebase archaeology, markup adaptations, the tenth rewrite of the same scraper — gets done by the machine. I spend my time on judgment calls: architecture, edge cases, the weird ones only a human notices.

And here is the uncomfortable corollary: if *I* can do this, so can everyone else. The implementation barrier that used to protect a good idea for months now protects it for days, maybe hours. Any competent developer with an AI agent can rebuild your "clever" solution over a weekend. The code was never the moat. It just took us twenty years and a language model to admit it.

## So what is the moat? The problem.

What the machine cannot do — still, and for the foreseeable future — is *choose the right problem*. It cannot live your life, notice the friction nobody else noticed, understand which annoyance is universal and which is just yours. It cannot decide that Greek streaming needs one addon instead of six, or that a file chooser portal deserves its own backend, or that Fermi GPUs deserve reclocking in 2026.

Taste is the moat. Problem selection is the moat. Domain knowledge, earned over a decade of broken antennas and dead repositories, is the moat. The "what" and the "why" — never the "how."

This reorders the craft completely. The valuable developer is no longer the one who can write the most code the fastest. It is the one who points at the right mountain while everyone else is busy optimizing their climbing gear.

## Which brings me to silence

If the idea — the problem selection — is the only scarce asset you have, then talking about it early is giving away the only thing you own.

I have learned this the expensive way. Mention an unfinished idea in the wrong channel and one of three things happens: someone with more free time ships a worse version first and now *you* look like the clone; someone with a bigger audience absorbs it without credit; or the idea gets debated to death by people who will never build anything, and you lose the will. All three have happened to me. I keep receipts.

So here is my advice, and I mean it as practical rather than cynical: **be stoic, and keep your ideas to yourself until they are materialized.** Not forever — shipping is the whole point — but until there is something real: a repo, a release, a demo, a post with substance. Ideas are weightless; execution has gravity. Nobody can steal a shipped thing. Everybody can steal a sentence.

This is not paranoia. It is arithmetic. When implementation costs days instead of months, the window between "I told someone" and "someone shipped it" collapses to nothing. The old rule — share early, get feedback — assumed feedback was scarce and implementation was slow. Both assumptions are now inverted: feedback is everywhere and worth little, implementation is fast and worth less. What is scarce is the unbuilt good idea. Guard it accordingly.

## How I actually work now

Since this sounds like fortune-cookie wisdom, here is the concrete version — my rules, such as they are:

1. **Ideas stay in private notes** until there is a working prototype. No teasers, no "big announcement coming," no screenshots of half-built things. The dopamine of announcing is a trap; spend it on building.
2. **Prototype fast with AI, judge slowly yourself.** The machine writes; I decide. If I cannot explain *why* this problem and *why* this shape of solution in two sentences, it goes back in the drawer.
3. **Ship, then talk.** The blog post comes after the release, never before. A shipped repo with a version number is a fact. A roadmap is a wish, and wishes get stolen.
4. **Share the how freely, guard the what carefully.** I will happily explain any technique, any trick, any debugging war story — the how is commoditized anyway. What I will not do is narrate my backlog. My technique posts are generous; my roadmap is silent.
5. **Assume good faith, verify with history.** Most people are not thieves. But it only takes one, once, to teach you the rule. Trust people who have shipped alongside you, not audiences.

None of this is about fear. It is about *order of operations*. Stoicism, in the original sense, is not about feeling nothing — it is about acting on what you control and ignoring the rest. You control what you build and when you reveal it. You do not control who is listening. Act accordingly.

## The craft is not dead. The scoreboard moved.

If you love programming — the actual craft of it — nothing about this era should depress you. The tedious parts got automated, which is what we always claimed we wanted. What remains is the interesting part: noticing, choosing, judging, finishing. The machine is a very fast intern with no taste. Taste is your job now, full-time.

Pick good problems. Build them quietly. Ship them loudly. And keep the next one to yourself until it is real.

That is the whole strategy. It fits on an index card, which is how you know it might be true.
