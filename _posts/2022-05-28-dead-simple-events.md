---
layout: post
title: Dead Simple Events in Monika After Story
date: 2022-05-28 17:19:35 +0200
categories: posts
tags: submodding
author:
  name: dreamscached
  picture: /assets/images/avatars/dream.png
read_time: true
image: /assets/images/posts/2022-05-28-dead-simple-events-hero.jpg
---

If you have never been working with MAS at level *way more advanced* than just some dialogue coding, you probably have
never wondered what is the system behind all of it...

Meanwhile, working on MSH ('MAS Self-Harm', get used to it, I'll often mention it like that because I like the
abbreviation :D) made me dig up the entire source tree and learn the aspects of it that are both repelling and
fascinating. So in that post, I'd want to make it *a bit more clear* how all of this works and what caveats you might
face working with events.

For instance, quick fun fact — Events can be both random (e.g. ones that Monika brings up on her own) and pooled (e.g.
those that player brings up, questions, compliments, etc) *at the same time*. Or that `unlocked=True` makes both types
appear in 'Hey, Monika' or 'Repeat conversation' as they don't differ too much? How about that, curious enough yet? :P

There's a lot more to it once you begin to discover what's using what, and how things are interconnected between each
other. In fact, *nearly everything* in MAS is made in events. Topics, menus, games, stories, songs, so on and so on.

Events were not changed since (*probably*) their very introduction (due to the specifics of how Ren'Py handles save
files and persistent) and are *extremely* simple in their model — they're merely objects with few properties that
(in complex, combined with each other) define when, why and how should something happen. And `random=True` and
`pool=True` is just the tip of an iceberg. Databases, conditionals, actions, start/end dates allow one to flexibly
control the behavior of that event and what it represents.

Enough of explanations though, how about some real-life examples? Starting with the *simplest* sort of events,
those that are just the most generic sort:

```python
init 5 python:
    addEvent(
        Event(
            persistent.event_database,
            eventlabel="generic_event_label"
        )
    )

label generic_event_label:
    m 1eua "Hi, [player]! Long time no see~"
    return
```

That's right, no prompt, no conditional, just an event label. That's the most basic type, mostly queued/pushed
programmatically and never unlocked, making it work like a one-shot dialogue that will only happen once:

```python
queueEvent("generic_event_label")
# ... as well as ...
pushEvent("generic_event_label")
```

Getting more advanced, these 'promptless non-unlockable' events can be random and appear randomly, but still just once:

```python
init 5 python:
    addEvent(
        Event(
            persistent.event_database,
            eventlabel="random_event_label",
            random=True
        )
    )

label random_event_label:
    m 1eua "I'm really sorry, but you won't see this one again..."
    return "derandom|no_unlock"
```

Can you spot the difference? Right, it has `random=True`, but then... something new — it has `"derandom|no_unlock"` in
its `return` statement. But what does it do? It's not so hard to have some anticipations by these return value names,
but this sure need some explanation:

* `derandom` sets `random` property of an event to `False`, meaning that this event will never appear randomly again
  on its own.
* By default, `random=True` events are unlocked when player reads through their dialogue. `no_unlock` prevents event
  from getting its `unlocked` property set to `True`, preventing it from appearing on the 'Repeat conversation' menu.

That combined with `random` makes event a one-shot topic! That's just how Monika's 'Thinking of first kiss' topic was
made (back then, it took me a bit to figure how to make it appear again... but that sure was a valuable experience :P)

Next up, let's examine more complex example — an event that will appear *if some conditions are fulfilled*. This is
where *actions* slam in, and once you figure how they work, you'll understand how much flexible events are! Let's
take a look at this one:

```python
init 5 python:
    addEvent(
        Event(
            persistent.event_database,
            eventlabel="random_conditional_event_label",
            conditional="player == 'Dream'",
            action=EV_ACT_RANDOM
        )
    )

label random_conditional_event_label:
    m 1eua "Hello, Dream! How's it going?~"
    return
```

You sure noticed that `random=True` is now gone. Yep! Until event's `conditional` fulfills, this event will have
`random` property set to `False` because it is the default value for it. Just until then! Once it fulfills,
`EV_ACT_RANDOM` is applied, and it sets `random` to `True`, giving this event an opportunity to pop up and (in this
case right here) greet *someone* whose name is Dream. *Maybe that'd be me :P*

It works *exactly the same* for pooling events, for unlocking them and queueing/pushing them based on conditions. For
other sorts of tasks, there are:

* `EV_ACT_QUEUE`
* `EV_ACT_PUSH`
* `EV_ACT_POOL`
* `EV_ACT_UNLOCK`

Alright, so while `QUEUE` and `PUSH` might sound quite straightforward (these are stupid simple! Basically, forcing
your event to appear, no matter what, given that conditional evaluates to `True` and gives it a greenlight), `POOL`ing
and `UNLOCK`ing events... *is not that simple.*

For starters, pooled events are by default *locked*. Yep, their `unlocked` is set to `False` (the default value, yea)
and they won't just magically appear on the topics menu right away. *However,* MAS has mechanism which is not mentioned
much often — *pooled events are unlocked **randomly.***

Yes, eventually, pooled events will get unlocked randomly and will make it into the 'Hey, Monika' menu. Been there
before! But thankfully, this is easily overridden:

```python
init 5 python:
    addEvent(
        Event(
            persistent.event_database,
            eventlabel="pooled_unlockable_event_label",
            prompt="How do you feel about caching dreams?",
            conditional="player == 'Dream'",
            action=EV_ACT_UNLOCK,
            pool=True,
            rules={"no_unlock": None}
        )
    )

label pooled_unlockable_event_label:
    m 1eua "Aw, I love it, Dream!"
    return
```

*Right, that's a bit of an introduction to the rules system.* `no_unlock` makes this event never get unlocked randomly,
only when your `conditional` works out or when you'll programmatically unlock it. That way you can be *sure* it works
as intended, and some topics only unlockable under certain conditions will unlock.

There are few other rules as well, but you'll mostly need just this one (stay tuned though, I'll explain the rest of
them further on, in future posts :D) for your everyday needs. For now, let's move onto one *critical* caveat of topics
which is likely a side effect of how events are stored and how their state is preserved for future sessions.

*Exhales.* Events are *unaffected by the addEvent calls* after they are once added. That should probably be *really*
obvious, but anyway, to me it was a surprise when I first stumbled upon it. So what does it mean?

Imagine coding an event, testing it (on your *dev install*, of course!) and then deciding to change something. Say,
its conditional. You make changes, save the `.rpy` file, start the game again... *but it doesn't change.*

Yep, that's *just* what I meant. Why does it happen though?

When events are used with `addEvent` calls for the first time, their state is *persisted*, and their properties are
now saved between sessions because they are now stored in *persistent file*. That also means that subsequent `addEvent`
calls will have no effect on them, and editing them will be... *problematic.* *Makes you understand why dev installs
are a necessity, right? :P*

Seriously though, yes, the only painless (or not so... depending on how you look at it!) way to change the events once
they're in persistent is to just discard it and bring a new one along. Yep, you gotta have a spare clean persistent
which wasn't populated with your variables and events yet.

Not so simple after all, am I right? But still, I can't stop getting impressed by how something *that simple* is being
used to create any sort of topic you like, and not just topics. Then again though — once you figure how it works
*under the hood*, it gets *almosts* clear. *Almost.*

Stay tuned, more posts about events, their nature and possible pitfalls are coming! Given that *I won't lose my
inspiration*, of course (thanks, Gaby <3)
