---
permalink: /news/on-delays-qtile-perfectionists-dilemma/
layout: post
title: "On Delays, Qtile, and the Perfectionist's Dilemma"
date: 2026-09-06 10:00:00 +0000
author: The AliveOS Project
---

Let us address the elephant in the room: AliveOS was supposed to ship v0.1 in
Q3 2026. That is not happening. The new target is **early Q1 2027**, and before
you close this tab in disgust, let me explain why the delay is not only
necessary but actually a sign that the project is getting *more* ambitious, not
less.

I could have slapped together an ISO, called it v0.1, and moved on. Many
distributions do exactly that. But I am a perfectionist, and shipping something
that is merely "functional" feels like a betrayal of the entire premise. If
AliveOS is supposed to be a *fast, coherent, no-compromise desktop*, then it
needs to actually be fast, coherent, and without compromise at launch &mdash;
not "good enough with a few known issues." There is a reason for that stance,
and it has to do with trust. More on that later.

So what is actually causing the delay? Three things.

---

## 1. Bug Fixing: The Unglamorous Foundation

![XConnect](/assets/icons/xconnect.svg)
{: .float-right }

The existing utilities &mdash; **xconnect**, **skript**, and the others
&mdash; are in that awkward stage where they *work* but do not yet *feel
right*. XConnect, the Android connectivity suite, has a few rough edges in its
notification syncing and media control that need smoothing. Skript, the
markdown viewer and editor, needs more work on its rendering pipeline before it
can be considered a daily driver.

This is the kind of work that does not generate exciting commit messages. "Fixed
edge case in notification parsing" does not exactly set the world on fire. But
it is the difference between a distribution that crashes when you look at it
funny and one that disappears while you work. I would rather ship late and
solid than early and fragile.

The bug fixing is ongoing and methodical. Every utility gets a thorough pass.
Every interaction gets tested. Every "that is technically correct but practically
annoying" behaviour gets fixed. This takes time. Not because the bugs are
particularly hard, but because finding them requires actually *using* the
desktop as a daily driver, which means I am both the developer and the QA
department. It is a lonely job, but somebody has to do it.

---

## 2. The Qtile Environment: Lightweight Done Right

This is the big one, and the reason the timeline shifted most significantly.

When I started AliveOS, the plan was a refined Cinnamon session. That plan has
not changed &mdash; Cinnamon remains the core. But somewhere along the way, I
started asking myself a dangerous question: *what if we also shipped an
extremely lightweight option?*

The answer is a **custom Qtile-based environment**. Not a tiling window manager
for tiling window manager enthusiasts &mdash; those people already know what
they want and have riced their setup six ways from Sunday. This is something
different. This is a **complete, opinionated, keyboard-driven desktop**
built on Qtile, with:

- **Custom apps** that fit the workflow &mdash; not GNOME apps awkwardly
  stuffed into a tiling layout, but purpose-built tools designed for this
  exact paradigm.
- **A control center** &mdash; because even a keyboard-driven desktop needs a
  place to adjust settings without remembering seventeen terminal commands.
- **A fast-paced workflow** &mdash; the kind where your hands never leave the
  keyboard and every action is one or two keybindings away. Not because mouse
  usage is forbidden, but because the keyboard path should always be faster.

Think of it as the *other* face of AliveOS. Cinnamon for people who want a
polished, traditional desktop. Qtile for people who want to fly. Both sharing
the same core apps, the same repository, the same philosophy &mdash; just
different window management paradigms.

This is a significant amount of work. Building a Qtile environment that feels
like a *desktop* and not a *window manager config* requires custom layouts,
custom widgets, default keybindings that actually make sense, and a control
center that does not feel like an afterthought. It is the kind of project that
starts as "how hard can it be" and ends with you rewriting the same widget
four times because the first three attempts were, in chronological order,
"functional but ugly," "pretty but unusable," and "why did I think this was a
good idea."

It is coming along. But it is not done yet, and I am not rushing it.

---

## 3. The Repository: More Pieces in the Puzzle

![Repos](/assets/icons/repos.svg)
{: .float-right }

The **aliveos-repo** custom pacman repository needs more packages. Not a
avalanche of packages &mdash; that would defeat the purpose of a curated
distribution &mdash; but a few more essentials that round out the experience.

This means building, testing, and hosting additional packages that fit the
AliveOS philosophy: light, fast, no unnecessary dependencies. Every package
that goes into the repository gets the same treatment as the custom apps: built
from verified sources, tested for conflicts, and reviewed before inclusion.

The repository is already functional and covers the core utilities. The delay
here is not about infrastructure &mdash; the CI/CD pipeline works, the build
process is solid, and the security scanner catches the obvious nasties. It is
about *curation*. Finding the right packages, building them correctly, and
making sure they play nicely with everything else. This is not a race. It is a
puzzle, and every piece needs to fit.

---

## The Perfectionist's Dilemma

Here is the uncomfortable truth: I could release an alpha right now. The core
works. The desktop boots. The apps function. It is usable. But "usable" and
"finished" are not the same thing, and I am stubborn enough to insist on the
latter.

The problem with shipping an alpha is expectation management. Once you put
something out there, people form opinions. They judge the project based on what
they see, not on what you *intend* to build. A buggy alpha becomes "that
half-baked distro" in the collective memory, no matter how good the final
release turns out. First impressions are sticky, and I would rather delay than
ship something that colours the project's reputation prematurely.

This is not about ego. It is about trust. If someone installs AliveOS and it
crashes, or a utility does not work, or the desktop feels rough &mdash; they
will not come back. They will tell their friends it was not ready. They will
write a forum post titled "AliveOS: Not Worth Your Time." And they will be
right, because at that point, it would not have been worth their time.

So I wait. I fix. I polish. I add the Qtile environment. I expand the
repository. I make sure that when v0.1 finally ships, it ships as something
that *works* &mdash; not something that mostly works with a README full of
caveats.

---

## The New Timeline

**Target: Early Q1 2027.**

That is the honest date. Not "Q3 2026 if everything goes well" &mdash; that
ship has sailed. Early Q1 2027 gives me the breathing room to finish the Qtile
environment, squash the remaining bugs, expand the repository, and do a proper
testing pass before putting an ISO out the door.

Is it late? Yes. Is it worth it? Also yes.

A delayed release is a memory. A bad release is a scar. I choose the former.

More updates as things progress. The next few months should be interesting.
