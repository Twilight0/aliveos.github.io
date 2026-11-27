---
permalink: /news/ai-ext4-patches-merge-window/
layout: post
title: "41,197 Lines at rc5: The ext4 AI Patch Flood and the Cost of Review"
date: 2026-11-27 10:00:00 +0300
author: The AliveOS Project
category: news
type: news
tags:
  - Linux Kernel
  - ext4
  - Artificial Intelligence
  - Code Review
  - Merge Window
---

On 24 November, Theodore Ts'o sent Linus Torvalds what looks like a routine `[GIT PULL] updates for 7.4-rc5` for ext4. Its second paragraph carried an unusual admission:

> The bulk of the new lines was caused by adding KUnit tests, which makes it safer to validate AI generated patches.

What followed is worth reading slowly rather than reacting to.

## What was actually in the pull

Three performance improvements — spreading sibling-directory inodes across flex-group block groups, moving orphan-list tracking off the journal's cache line, skipping holes when shrinking the extent tree during fast-commit replay — and then, in Ted's own words, *"a huge number of bug fixes, most of which were AI generated"*:

- fast-commit fixes preventing loss of fsync'd data after a journal flush
- checking `blkdev_issue_flush()` return values in ext4 and jbd2
- `EXT4_IOC_MIGRATE` data loss on contiguous ranges beyond `EXT_INIT_MAX_LEN`
- out-of-bounds accesses in inline-data and in-inode xattr handling on corrupted filesystems
- use-after-frees in discard work and in `ext4_split_convert_extents()`
- a buffer/folio lock inversion deadlock in `__ext4_get_inode_loc()`
- readdir position truncation on 32-bit kernels above 4 GiB

Plus KUnit regression tests for the inode migration fixes — the part that drew the sharpest objection.

## In the right hands, this is genuinely impressive

Set the timing aside for a moment and look at the work itself. These are not style nits or warning cleanups. They are data-loss bugs, use-after-frees and deadlocks in one of the most load-bearing filesystems on earth, and someone produced a large batch of them, with tests, in a cycle.

That is the case for AI-assisted development in its strongest form: the boring, high-diligence work — reading error paths nobody has read in years, noticing that a return value goes unchecked, spotting an extent range arithmetic overflow — gets done, and done quickly. In the right hands it is a multiplier on attention, not a replacement for judgement. Ted's pull is a real demonstration of that.

## But the review burden is the point

Linus did not dispute a single fix. He disputed the timing:

> Honestly, I don't think this was appropriate for this stage. You should have kept your tests in a separate branch for your testing, and done this *huge* update for the next merge window.

His central warning is one the AI discourse keeps skipping:

> STOP THINKING THAT YOU — OR AI — NEVER ADD BUGS WHEN YOU FIX SOMETHING.

His reasoning is precise. If the bugs were subtle and had survived for years unnoticed, then the fixes are, by definition, addressing something subtle and rare — and *"that makes the fixes possibly buggy."* A tool that finds the hard bugs is a tool that writes the hard fixes. The difficulty does not vanish; it relocates from the author to the reviewer.

## ext4, and the timing

Two aggravating factors made this worse than a routine style disagreement.

**The subsystem.** This is not a driver for exotic hardware that a handful of people run. ext4 is the default filesystem for most Linux distributions; a regression here corrupts users' data on a planetary scale. Reviewing a fix in ext4 means holding the mental model of journal replay, fast commit, extent trees and the block layer in your head at once — and now doing it for 41,197 new lines at a sitting.

**The stage.** The merge window had closed weeks earlier. We were at rc5 — precisely the period Linus describes as *"several weeks of calming things down."* His objection to Ted was blunt about the process:

> This whole notion of "anything that is a fix goes in -rc" is bogus.

And the cost is not theoretical:

> If you think that "41197 insertions" is ok and cannot introduce new problems just because most of it won't be compiled or because they were fixes for other things, you are full of crap.

The correct venue for AI-generated volume is the merge window, where conflicts are cheap and there is a full cycle of rc testing ahead. Dropping it mid-rc forces the maintainer to resolve conflicts in test infrastructure he did not write, under deadline, without running any of it.

## How it ended

Ted offered to unpull and re-send without the tests:

> The tests were on a separate branch, and I deliberately put it at the end so if you had objected, I could very easily drop them from the pull request... So if you want to unpull I can send you a revised pull request and I'll save the tests for the merge window.

Linus had already merged it, and kept it — but the closing note is the most honest sentence in the whole thread:

> The test resolutions look "ObviouslyCorrect(tm)" to me, but I didn't *run* any of them, so they may be complete garbage.

That single line is the entire problem in miniature. The merge is done, the conflict resolutions are plausible, and nobody has executed them. The maintainer absorbed the risk so that the cycle would not slip.

## The takeaway

Nothing here argues against AI-assisted kernel work — the fixes are real and valuable, and the KUnit tests that caused the objection are exactly the right instinct: they are what makes generated patches verifiable at all.

The lesson is about *who pays for the verification*. AI moves effort from writing to reviewing, and reviewing is not free, not automatable to zero, and hardest in precisely the places where mistakes cost the most. The right workflow keeps human judgement at the decision points and gives the volume somewhere safe to land — a separate branch, merged during the merge window, where a full cycle of release candidates exists to catch what the reviewer missed.

*Source: [LKML thread — [GIT PULL] updates for 7.4-rc5](https://lore.kernel.org/lkml/awW86EI5fYafQx-o@mit.edu/)*


