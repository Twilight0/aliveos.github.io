---
layout: post
title: "Summer, Bugs, and a Slight Change of Direction"
date: 2026-07-24 08:00:00 +0000
author: The AliveOS Project
---

It has been a while since the last update. Not because the project is dead.
Far from it. But because life happened, as it tends to do, especially in
summer. Let me explain.

## What I was actually doing

Bug fixing and planning. The unglamorous stuff that does not make for a
thrilling blog post but is absolutely necessary if you want a distribution that
does not fall apart on first boot.

I have been working on AliveOS's main applications &mdash; **xconnect**, **Dory**,
and the rest of the gang. Polishing. Refactoring. Asking myself hard questions
about whether each piece actually earns its place. The usual.

This is the part of open source that nobody talks about in their README: the
long, boring slog between releases where you stare at a log file for an hour,
change three lines of code, and call it progress. Glamour.

## The summer problem

Here is the thing about balancing a project like this with real life: it is
hard. Summer makes it harder. There is family, there is work, there is the
endless tug-of-war between "I should fix that bug" and "I should go outside
before I turn into a pale hermit with excellent git hygiene."

I am trying to catch up with the existing work-life-family balance, and it is,
shall we say, a work in progress. The project moves forward, but not at the
breakneck pace one might hope for. Steady is the word. Like a glacier, if
glaciers wrote code and occasionally forgot to eat lunch.

## The focus shift

Now, the part that actually matters.

When I first started AliveOS, I talked a lot about a **developer-optimized**
environment. That was the framing, and it was honest as far as it went. But the
more I worked on it, the more I realized the label was too narrow, and frankly,
a bit misleading.

The real goal of AliveOS was never about catering exclusively to developers.
The real goal is something broader and, I think, more interesting: **a
fast-paced workflow that targets general (and development) desktop usage,
fusing keyboard and mouse driven paradigms.** Essentially, what Cinnamon
can already do &mdash; but refined, tuned, and integrated so it stops being a
collection of features and starts being a *flow*.

A desktop where you never have to lift your hands from the keyboard unless you
want to. But also a desktop where the mouse is not an afterthought, and where
every action has a path &mdash; keyboard, mouse, or a combination of both, and
you pick whichever is faster for the task at hand.

This is not about being a "developer distribution." This is about being a
**fast desktop** for anyone who cares about speed, regardless of whether they
write code, edit video, or just manage five hundred open browser tabs like a
civilized lunatic.

## Design DNA: borrowing from COSMIC and Unity

You might look at the direction and wonder where the visual and interaction
ideas come from. I will be honest about that, too.

The design of AliveOS intentionally fuses concepts from two sources that, on
paper, should not have much to do with each other: **System76's COSMIC** and
**Ubuntu's Unity**.

From COSMIC, I take the idea of a clean, focused, no-nonsense interface that
gets out of your way. The way it handles workspaces, the attention to
consistent spacing and typography, the feeling that somebody actually thought
about where every pixel goes &mdash; that is the kind of polish I want, without
the Plasma-level complexity hangover.

From Unity, I take the keyboard-and-mouse fusion that was ahead of its time and
still, in my opinion, unmatched for workflow density. The HUD, the global menu,
the way Unity let you drive the entire desktop from the keyboard without
forcing you into a tiling window manager straight out of a 1990s hacker movie.
Unity understood something that many modern desktops forgot: you do not need to
choose between keyboard and mouse. You use both, fluidly, and the desktop
should keep up.

AliveOS is not a clone of either. It is an attempt to take the best lessons from
both &mdash; COSMIC's visual discipline and Unity's interaction model &mdash;
and fuse them into something that runs on Cinnamon's solid, resource-conscious
foundation. Borrowing good ideas is not cheating. It is standing on the
shoulders of giants who got it mostly right.

## Why "developer-focused" was the initial frame

Good question. The answer is boring and honest: because that is the lens I was
looking through at the time.

I am a developer. I was building what I knew I needed. The developer focus was
a natural starting point &mdash; it reflected the person building it, not the
audience it was meant for. The mistake was assuming that "what I need" and
"what the distro should be about" were the same thing. They are not quite the
same.

The truth is that a fast, keyboard-driven workflow with sensible defaults and
no bloat is useful for *everyone*, not just people who run `gcc` for a living.
The developer angle was a useful initial sketch. But the real painting is
broader. It is a general-purpose desktop that happens to be excellent for
development &mdash; rather than a development desktop that happens to function
for other things. The order matters.

## Where we are heading

So, the direction is not changing drastically. It is clarifying. Fewer "for
developers" labels, more "this is fast and you will feel it" reality. The same
core philosophy (zero bloat, sensible defaults, Cinnamon's sweet spot) just
with a more honest description of what we are actually trying to achieve.

More news as things land. After me.

> Zero Bloat Policy reminder: fast does not mean fat. The goal is a desktop
> that stays out of your way, whether you are coding, designing, or just trying
> to open a file before your afternoon coffee kicks in.
