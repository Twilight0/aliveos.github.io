---
layout: post
title: "Cinnamon Merges Are Slow, Users Don't Care Why, and Why I'm Toying With the Idea of a Fork"
date: 2026-09-14 10:00:00 +0000
author: The AliveOS Project
---

Let me tell you how open-source desktop politics actually works in the real world. Not in theory. In practice, on your machine, when something is broken.

Something breaks. You, the user, do not care whose fault it is. You do not care that AliveOS ships Cinnamon from upstream, that the applet came from the Spices collection, that the maintainer lives in a different timezone and has a different release philosophy. You know one thing: Twilight0 gave you this system, Twilight0's name is on it, so Twilight0 owns the bug. And you are right. That is how it should be. The person who hands you the ISO owns your experience, full stop.

Which brings us to the problem.

## The queue where good fixes go to wait

![Spices backlog](/assets/icons/puzzle.svg)
{: .float-right }

Upstream Cinnamon review is slow. Not slow as in "careful and methodical." Slow as in fixes that crash on every single use sit in the open queue for weeks while exactly nothing happens. No comment. No review. No merge. Just silence.

Concrete example, my own, no hearsay: [spicy-clipboard-applet PR #9008](https://github.com/linuxmint/cinnamon-spices-applets/pull/9008), opened August 31st. What does it do? Three things, all obvious, all small:

1. It fixes a crash. The applet called `Gio.File.make_directory_with_parents_async`, a function that does not exist in the GIO API. Every copy operation threw an unhandled `TypeError`, and clipboard history never got saved to disk. That is not a feature request. That is broken software doing the one thing it claims to do.
2. It stops `xclip` subprocess leaks. The applet polls the clipboard on a 300ms timer. If one `xclip` check stalls, the next tick spawns another one, and another one, until you are exhausting Xorg client connections. The fix is a re-entrancy guard flag. One boolean. This is the kind of bug that eats your session alive.
3. It adds a configurable keyboard shortcut to open the clipboard menu, folding in another contributor's work (#8975) that was also sitting around.

The whole thing is 3 commits, 2 files changed, 77 lines added, 12 removed. Mergeable state: clean. And as of today, two weeks later: zero comments, zero reviews, still open. A crash fix plus a resource-leak fix plus a requested feature, in a tiny diff, reviewed by nobody.

Now multiply this by every applet, desklet, extension and action in the Spices repos, and then ask yourself a simple question: how is a downstream project supposed to ship "a working, bug-free system" on top of this? The answer is, it cannot — not if it waits politely in line. Users will not file their anger at the right repository. They will file it at me. And they should, because I shipped it to them.

So here is an idea I keep coming back to — emphasis on *idea*. Nothing decided, no repository, no timeline, no announcement. Just thinking out loud.

## The idea: a fork that would be faster, saner, and done properly

![Cinnamon](/assets/icons/cinnamon.svg)
{: .float-right }

The thought, as floated back in July, is this: *what if* there were a Cinnamon fork that moved at the speed of actual problems? Not a hostile fork. A practical one. Hypothetically, what would that even mean in plain language?

**Fast adoption of obvious fixes.** A crash fix with a clean diff does not need a month of contemplation. It needs someone to test it, and then it needs to land. Such a project *would* use AI assistance aggressively for exactly this: triage, impact analysis, regression-risk scoring, test-matrix suggestions, rebasing help. Not AI slop merged blindly — AI doing the mechanical 80% so humans can spend their minutes on the judgment 20%. The current process spends human minutes on nothing at all, which is worse than any AI risk you can name.

**Real features, not fashion.** Here is one: optional, local AI integration. Opt-in, on-device, private, no cloud nonsense. Summarize a document, find a setting, automate a repetitive desktop chore — your business stays on your disk. If you do not want it, you turn it off and it is gone, not lurking. Innovation, imagine that, in a traditional desktop.

**GTK3/XApp integration done right, everywhere.** Cinnamon's strength was always coherence: the panel, the file manager, the dialogs, the settings, all speaking the same visual language. Lately the fashion is to bolt on GTK4 counterparts here and there, or worse — and this one genuinely takes the cake — to *re-skin GTK3 dialogs so they merely look like GTK4*. Form over function. Lipstick on a toolkit. Absolutely ridiculous. A hypothetical fork could do the unfashionable thing instead: take GTK3 and XApp widgets seriously, tighten them up in every corner of every app, consistent theming, consistent behavior, no half-migrated Frankenstein toolkit mix where every second dialog looks like it wandered in from a different operating system.

## The file manager test

![File manager](/assets/icons/dory.svg)
{: .float-right }

And here is where I get properly dedoimedo about it, because this is the test that matters and almost every Linux desktop fails it.

Open a file picker. Save a document. Attach a file to an e-mail. Plug in a USB stick. On Windows and macOS, this works the same way every single time: same dialogs, same places, same behavior, portals and pickers agreeing with each other. Boring. Predictable. Correct.

On Linux? Roll the dice. Native dialog here, portal dialog there, GTK3 here, GTK4 there, thumbnails present on Tuesdays, missing on Wednesdays. The file manager says one thing, the open dialog says another, the save dialog says a third. Users notice. Users always notice. They may not know the word "portal," but they know the feeling of "this system does not trust itself."

If such a fork ever happened, it could fix this end to end: one file manager, one portal story, one set of dialogs and pickers that behave consistently in all cases. Not exciting. Not demo-ware. Just the thing that separates a toy from a tool. The highest-tier operating systems figured this out decades ago. Time someone did too.

## Bottom line — and to be clear

Slow merges are not a personality flaw, they are a structural fact: small upstream team, huge surface, stability-first philosophy. I understand it. But understanding it does not fix my users' clipboard applet, and my users are not wrong to hold me responsible.

So this whole fork notion is just that — a notion. An idea I am turning over in my head, nothing more. There is no fork repository, no plan with a date on it, no team assembled. If my name is on the system, then I owe you thinking about how the fixes *could* land on a saner schedule: fast review with AI doing the heavy lifting, honest GTK3/XApp coherence instead of GTK4 cosplay, local AI you can switch off, and file dialogs that behave like they were designed by adults.

Users expect a working system. They are right to expect it. For now, I am doing what a downstream can do: patch, test, and upstream what I can. The rest is thinking out loud.
