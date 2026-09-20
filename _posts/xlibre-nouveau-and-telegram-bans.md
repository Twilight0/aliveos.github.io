---
layout: post
title: "Xlibre, Nouveau, and the Art of Getting Banned from Telegram"
date: 2026-09-15 12:00:00 +0000
author: The AliveOS Project
---

Let me tell you a story about enthusiasm, skepticism, and getting banned from a
Telegram channel for the crime of mentioning AI. But first, let us talk about
Xlibre and why I am not entirely convinced it is the promised land.

## The Xlibre Situation

Here is the thing about Xlibre: it *should* be perfect for a project like
AliveOS. Legacy hardware, legacy drivers, a display server that does not
require a PhD in Wayland compositor archaeology to configure. The pitch writes
itself. A distribution that supports a broad spectrum of hardware, combined with
the rock-solid foundation that Xlibre offers &mdash; what is not to like?

Well, quite a bit, actually.

I have been watching Xlibre with genuine interest. I have an old laptop with
hardware that predates the modern "everything must be Wayland or die" era. Xlibre
looked like the natural home for it. But the more I look, the more I see a
project that is moving fast without quite knowing where it is going, and that
makes me nervous.

### The bug situation

Xlibre has bugs. Lots of them. Not the kind of bugs that you find in
cutting-edge software and shrug off as "growing pains." The kind of bugs that
make you wonder whether anyone is actually testing this before shipping it.

I am not talking about the occasional regression that slips through. Every
project has those. I am talking about a pattern &mdash; a steady drumbeat of
issues that suggest the development process is optimised for velocity over
stability. Which is a choice, I suppose, but not one I particularly want in a
display server that is supposed to be the foundation of my desktop.

### The code of conduct situation

Here is something that tells you more about a project than any changelog: the
absence of a code of conduct. Xlibre does not have one. In 2026.

Now, I know what some of you are thinking: "Codes of conduct are just virtue
signalling." No. They are not. A code of conduct is the minimum viable
agreement that says "we are building something together, and here is how we
behave while doing it." It is the architectural drawing for human interaction.
Its absence does not mean the project is dysfunctional &mdash; but it does mean
there is no agreed-upon standard for when things go sideways. And in my
experience, things always go sideways eventually.

### The Arch Linux silence

It is no surprise that Xlibre is absent from the Arch Linux Wiki. The Arch
community is pragmatic to a fault &mdash; if something works, it gets documented.
If it does not work, or if the jury is still out, it does not. The silence
speaks louder than any forum post.

## The Artix Linux Detour

Speaking of things that speak louder than words, let us talk about Artix Linux
and their recent return to Xorg.

Artix, for those unfamiliar, is an Arch-based distribution that offers
OpenRC, runit, and s6 init system options. They have been sympathetic to Xlibre
for a while, and for a while it seemed like a natural pairing &mdash; a
distribution that values simplicity and choice, embracing a display server that
values the same.

Then they went back to Xorg.

The reasons are not entirely clear, but the signal is loud: Xlibre is not
ready for prime time, at least not in Artix's estimation. When a distribution
that explicitly chose *not* to use systemd decides that your display server is
too much trouble, that is not a ringing endorsement.

### The Telegram incident

Now, the part that actually pissed me off.

I have been working on improving nouveau &mdash; the open-source NVIDIA driver
&mdash; because I believe that AliveOS should offer good GPU driver support for a
broad spectrum of hardware. The modern equivalent of hardware being expensive
due to AI makes this even more important. If you cannot afford new hardware,
you had better make sure the old hardware still works.

I have been using AI tools to help with this work. Specifically, I have been
using AI to reverse-engineer mmiotrace scripts that can help with frequency
reclocking for hardware that nouveau does not fully support yet. Memory
reclocking sort of works now, which was the biggest hurdle. The frequency
reclocking is next.

I mentioned this in the Xlibre Telegram channel. Not as a provocation, not as
a political statement, but as a factual update on what I was working on. "Hey,
I am using AI to help improve nouveau, here is what I have found."

I was banned. No warning, no explanation, no "please don't discuss that here."
Just gone. One moment I was a member of the community, the next I was a
ghost.

For the crime of mentioning AI.

Let that sink in. In 2026, in a community that is supposedly about open-source
software and collaborative development, I was banned for using a tool that
helps me write better code. The irony of a project that forked from X.org
because of governance issues deciding to ban people without governance is not
lost on me.

## Where This Leaves AliveOS

So, where does this leave us?

I am sceptical about Xlibre. Not hostile, not dismissive, but sceptical. The
bugs are real, the governance is unclear, and the community management
(present company excluded) leaves something to be desired.

I am also pragmatic. Xlibre has genuine advantages for legacy hardware, and
ignoring those advantages because of politics would be stupid. But adopting it
as the default without acknowledging the risks would also be stupid.

Right now, my plan is this:

1. **Keep improving nouveau.** The work I am doing with AI-assisted reverse
   engineering is paying off. Memory reclocking works. Frequency reclocking is
   next. This benefits everyone, not just AliveOS users.

2. **Watch Xlibre, but do not bet the farm.** If it stabilises and the
   governance matures, great. If not, we have alternatives.

3. **Cater to Wayland users too.** The future is not exclusively X11, and
   pretending otherwise would be doing our users a disservice.

4. **Stay true to the zero bloat philosophy.** Whatever display server we
   use, it has to earn its place. The same goes for every other component.

The goal of AliveOS has always been to provide a fast, stable, no-nonsense
desktop that works on real hardware. That means supporting the hardware people
actually have, not just the hardware companies want to sell them. And that
means being willing to use whatever tools get the job done &mdash; including AI,
if that is what it takes.

If that gets me banned from another Telegram channel, so be it.

> Zero Bloat Policy reminder: a display server should display things. That is
> the entire job. If it cannot do that reliably, it does not matter how
> philosophically pure it is. Stability first, ideology second.
