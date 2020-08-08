---
title: "Negotiating with the gatekeepers"
description: Proper scrutiny is important, but CABs are an inefficient and ineffective way to scrutinize.
author: alex.yates@dlmconsultants.com
visibility: public
published: 2020-07-15
metaImage: 
bannerImage: 
tags:
 - DevOps
---

“No matter what they tell you, it’s always a people problem.”
Gerald Weinberg

This blog post is not about software, or automation, or processes. Today I choose to write about people.

Last month [I wrote about the ineffectiveness of Change Advisory Boards (CABs)](https://octopus.com/blog/change-advisory-boards-dont-work). I didn’t hold my punches when discussing the harm caused by bureaucratic change approval processes or unhealthy attitudes with risk. I stand by every word.

Afterwards, on Twitter, a few folks asked me how to handle pushback from various gatekeepers, so today I’d like to say something about that. But before we go any further, I’d like to try something a little unusual.

Dear reader, I’m asking you to take a leap of faith with me.

Here goes…

~ ~ ~ ~ ~ ~ ~

8am.

You’ve only got one coffee down you. You take a sip from your second and replace it in the cup holder. 

It’s sour. Apparently, you were in a rush this morning. It’s hardly satisfying, but you decide to force the rest down anyway. You’ll need it.

Pitter-patter. Wish-wash. 

Two lanes are shut. You aren’t going anywhere fast. They’ve been digging holes in the road for months, but you don’t see any sign of progress.

The irritating “pitter-patter, wish-wash” of the weather and your wipers is itching at whatever strands of patience you’ve been using to hold yourself together.

Pitter-patter. Wish-wash. 

You’ve been up half the night because some “rock star” ran a script on production at 6pm last night without telling you – and they made a stupid mistake. You were sat in the traffic on the other side of the road when your phone notified you that something was seriously wrong, and you’ve been dealing with the fallout ever since.

This isn’t the first time, but your boss still expects you back in the office at 9am, sharp.

At first you dealt with these issues calmly and proactively. You attempted to help the rock stars to understand the issues, but they weren’t interested because “that’s DBA stuff”.

Pitter-patter. Wish-wash. 

You’ve learned to accept the sleep deprivation as routine. What’s really getting to you is that you didn’t read your little boy his bedtime story. You haven’t seen him at all in 24 hours. You miss his cheeky grin and can’t help resenting that some trigger-happy developer has stolen those precious moments from you. You feel like a bad parent.

You try to put that to the back of your mind. It’s not professional.

Pitter-patter. Wish-wash. 

Your boss is seething. You were offline for several hours and you’ve lost a small amount of data. A few dependent services were also affected so your boss is getting heat from various other managers who possess more political power than technical ability. 

It’s time to face the music. “How did YOU let this happen?” “Why did YOU take so long to bring everything back online?” Folks are out for blood.

There’s a retrospective (blame circus) first thing. You need to be there, but you’re shell of yourself.

It’s the end of November - now is not the time to be losing your job. Your thoughts return to your boy, and the idea of cancelling Christmas. Too much.

Pitter-patter. Wish-wash. 

Rain. Brakelights. Bad coffee.

You glance at your watch. You realise that you might be late and feel a spasm of panic shoot up your spine. No-one is even working in those pointless looking holes in the tarmac!

You pull yourself together. You are respected at work for your calm diligence and professionalism. You’ve got this.

Breathe. Count to ten.

One… Two…

9.05am.

“OVER MY DEAD BODY!” you hear yourself scream. Your last thread of patience snapped and you’re no longer in control.

This retrospective could have gone better.

“Be reasonable!” says the rock star. Patronisingly. “We never have these issues with app deployments because we do them through Octopus. If we could release on demand, I wouldn’t need to go behind your back.”

Rock star was smiling as the words came out. They were directed at you, but designed for the VIP spectators. Conniving little…

You can feel the anger boiling over. The next few minutes don’t go well.

~ END SCENE ~

A little empathy can go a long way.

I’ve been that “rock star” before. I didn’t understand the gatekeepers. I saw them as bottlenecks. I might have tried to be a little politer than the characters above, and I’ve never had any time for politics (sometimes to my detriment), but I admit that I probably came across as young, naïve and cocky. At some level, I probably did see those gatekeepers as dinosaurs. I probably did feel like I knew better. And they probably had a point.

I see a lot of people with “DevOps” or “SRE” in their job title behaving in the same way. This comes across as patronising and is more likely to stoke divisions than unify silos. Especially when speaking to veteran engineers, who have spent their lives bravely battling to keep those fragile servers running without ever having had either a boss that understood the problems nor the resources they needed to fix things properly.

The truth is, I’d read different books, I’d been to different conferences, and I’d worked in companies with different values. The gatekeepers didn’t even have Twitter accounts…

I’m a big fan of Simon Sinek’s “Start with Why”. If you haven’t watched his TED talk, [bookmark this 18 minute video](https://www.ted.com/talks/simon_sinek_how_great_leaders_inspire_action?language=en) for your next coffee break.

Sinek’s basic idea is that before suggesting specific plans, such as automating the production deployments or scaling back the CAB, you need to show people, sincerely, that you believe in the same things. 

Sinek jokes about how Martin Luther King Jnr never gave the “I have a plan” speech. It wouldn’t have had anything like the same appeal. Folks might disagree on the best plan of action, but if they believe you share their dreams, they might be willing to give you half a chance.

If you want to build a bridge across the “[wall of confusion](https://levelup.gitconnected.com/the-wall-of-confusion-623057a4dd26)” between development and operations, forget about your current petty disputes. Try to meet with the gatekeepers somewhere off the record and less formal than the boardroom and talk to them about the bigger issues that really concern them. You’ll probably find that you have more in common than you thought.

After all, you are both in the business of getting stuff done, without breaking things. And neither of you want to get fired - especially not just before Christmas.