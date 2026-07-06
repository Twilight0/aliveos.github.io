---
layout: post
title: "How It All Started: One File Picker, a Rejected PR, and a Detour Through KDE"
date: 2026-07-06 19:00:00 +0000
author: The AliveOS Project
---

Every distribution has an origin story. Most are boring. "We wanted a simpler
Linux," they say, and then ship forty gigabytes of snaps. AliveOS's story is
not that.AliveOS begins not with a grand vision, but with a single annoying dialog box.
Follow me.

## The something that was missing

So, there I was, on Cinnamon, doing my thing. Happy-ish. And then I needed to
pick a file. The dialog that popped up was, to put it kindly, *not on par* with
Dolphin's. Dolphin gives you a rich picker: zoom in, zoom out, switch view
modes, a whole bag of options at your fingertips. The XApp desktop portal that
Cinnamon relied on for this duty was, by comparison, spartan. Functional, sure.
Pleasant to use, absolutely not. A relic. A thing that felt like it time-traveled
here from 2008 and was mildly proud of it.

And when you spend your day opening and saving files, that gap is not cosmetic.
It is a real, daily, morale-eroding annoyance. The kind you ignore for a year
and then one day you snap.

## A PR, with help from an unlikely collaborator

I wanted the functionality badly enough to do something about it. Stubbornness
is an underrated engineering virtue. So, with the help of AI &mdash; yes, the
very thing everyone is either worshipping or panicking about this year &mdash;
I put together a pull request against Nemo's upstream source repository. The
changes were real: zoomable, configurable, closer to the picker I
actually wanted to use. Written. Shipped. Not just a rant on an issue tracker.

And then it got rejected.

## "We can't push this to other distributions"

The reasons given were, shall we say, vague. The gist: they couldn't push Nemo
changes onto other distributions that currently depend on XApp. Which is the sort
of rejection that sounds reasonable until you think about it for roughly thirty
seconds &mdash; because for file *picking*, you can comfortably use the XApp
desktop portal independently from the file manager itself. The two are not
coupled the way the objection implied. Not even close.

That rejection sat with me. I had the change. I had the use case. I had nowhere
to put it upstream. Ipso facto, I was grumpy.

## The detour through KDE

So I did what any stubborn person does: I went somewhere I *could* have it. I
migrated to KDE and worked on it for a month, moving away from Cinnamon &mdash; a
desktop I had been using for the better part of a decade. Now, KDE is genuinely
good. I like KDE. I will get to that. But then came the small matter of KDE
dropping X11 for Wayland.

And there was the catch. I have a really old laptop. I am not changing to Wayland
anytime soon. "Modern" does not impress me when it stops working on my hardware.
And for me, **XLibre** looks like the more promising path forward on this
venerable old box.

So I came back. And this time, rather than asking permission from anyone, I
decided to put together a few pieces I find useful and straightforward. A
desktop that finally has the picker I want, on a base I trust.

## Why not just bolt on a few apps?

Ah, the obvious question. It would be easy, in theory, to just layer a few
programs from other projects on top. Grab `gnome-calculator` here, something
else there. Job done, innit.

No. The problem is twofold. One, these bring external dependencies that add
bloat. Two, they look inconsistent with the rest of the system &mdash; like you
parked a Fiat in the middle of a BMW lot and pretended nobody would notice. Two
small things that, together, defeat the point of running a coherent desktop in
the first place.

## The sweet spot

Now, here is a thing I have learned over many years. I value the balance between
three things: aesthetics, functionality, and resource usage. Most desktops pick
one and sacrifice the other two. For me, **Cinnamon sits exactly on top of that
sweet spot.** That is why I have been using it for almost ten years. Which, in
Linux desktop years, is roughly three geological epochs.

Before that, I was a GNOME user. I have also used KDE. And the honest truth is: I
like all of them. Every desktop environment and window manager has its pros and
cons, and I genuinely appreciate each one for different reasons. Heresy, I know.
A grumpy dinosaur is supposed to pick a side and hate the rest. Sorry to
disappoint.

AliveOS is not a rejection of anything. It is a curation &mdash; the specific
pieces, tuned to work together, on a base I trust, built around a picker that
finally does what I want it to. After me.

> Zero Bloat Policy reminder: this is exactly why the policy exists. Coherence
> beats quantity. The right pieces, working together, beat a larger pile that
> doesn't. You get the polish, not the pile.