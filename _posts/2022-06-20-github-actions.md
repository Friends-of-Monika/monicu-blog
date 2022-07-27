---
layout: post
title: GitHub Actions For Submod Checking
date: 2022-06-20 09:06:02 +0200
categories: posts
tags: submodding github repo
author:
  name: dreamscached
  picture: /assets/images/avatars/dream.png
read_time: true
---

Continuing the series of posts about *more or less* advanced submodding 
experience, today I'm going to talk about GitHub Actions, and how they can
assist you with your submod repository organization, releasing and code 
checking.

Helping Otter with her [MAS Self Harm Submod][1], one of the first things I
thought of adding was code checking, in order to spot my own (and other
members' too!) bugs, syntax flaws and other possible problems right on commit
&mdash; without a need to manually check the source tree every time someone
pushes new changes. More than just that, I was still impressed by the potential
of Actions and CI/CD overall; especially, with the possibility of using `main`
and `dev` branches as `stable` and `unstable` channels, with an option to create
new releases on merge from `dev` into `main`.

What you can see at the self-harm submod repository now is my *first* take at
achieving the goal I've set for myself; but it definitely wasn't the last one, 
as one thing this solution lacked was *readability*, *configurability* and 
*difficulty* to reuse even on my own, let alone someone *really* new to GitHub. 
Besides figuring how to set the entire workflow up, one had to set up their own 
file server to pull DDLC package from (remember about 
[Team Salvato IP Guidelines][3], right?) and if this approach to *private*
DDLC package hosting was too complex or for whatever reason inappropriate to
user... That would be just too complicated overall, taking a way to write a
script to pull it the other way from the ground up. So I left it as it is,
unchanged. If it works, it works, right?

My next attempt was to create a submod template that could include all the setup
instructions &mdash; but this one wasn't as successful either, as I couldn't get 
satisfied with my own explanations and instructions, and looking at it from
someone else's point of view, it was *too Linux and command-line-centric*, still
making things more complex and less comprehensible to new people.

Then, [composite actions][2] came to rescue me here &mdash; and we're just what
I wanted and needed. Taking parts from original workflow used in self-harm
submod, combining them with refined and simplied pieces of submod template code,
I created [an action][4] that could be used easily, without a need to set up 
*almost* anything &mdash; the only thing one needed to decide is how to provide 
a DDLC package (and currently, this can be easily done with uploading DDLC .zip 
to one's Google Drive and storing its share link as repository secret.)

To see how *really* simple it is, you can peek into [submod-check-example][5]
repository I've initially made just for testing it, but decided to keep it as
an example to those who wish to use it with their submods too.

Everything boils down just to creation of simple workflow file, Google Drive
upload and setting repository secret with the shared link. And your code check
workflow is ready! If you're more advanced, you can go a bit further with it
and configure automatic release creation, or anything!

Possibilities are *indeed* endless with Actions.

[1]: https://github.com/my-otter-self/mas_selfharm
[2]: https://docs.github.com/en/actions/creating-actions/creating-a-composite-action
[3]: https://teamsalvato.com/ip-guidelines
[4]: https://github.com/friends-of-monika/submod-check-action
[5]: https://github.com/friends-of-monika/submod-check-example