## Intro

Back in 2018, where I worked there was an operations team. "Ops", we called them.
In that decoade, this company was behind the curve, but maybe typical. We were just starting to _think about_ AWS, and near the tail end of my time there, just starting to adopt it for some internal-only systems. But from what I've heard, it wasn't uncommon in that era to have _an operations team_ that owned production.

Funny thing: the ops team literally sat in a corner of the office, in their own room. That's where ops is, in that little room. It sounds like a meme.

The ops team had a nice tool to spin up a VM inside the company's infrastructure. I appreciated that -- my whole team used it all the time. I needed to train [recurrent neural networks](https://en.wikipedia.org/wiki/Long_short-term_memory) using GPUs and 20+ gigabytes of RAM. No way that was going to run on my laptop, so this workflow was invaluable to my work.

Here's the big catch: production deployments happened once every two weeks. Full stop.

If something went wrong, the deployment had to wait another two weeks.
Unless you were lucky: if the current ops on the weekly rotation was particularly nice, and not dealing with evening plans, and if you were online to respond to their questions, you could push through and fix that random error that only happens in production.

From time to time, I would wander into the ops corner and chat with people about strange issues my team saw in the production database, in our latest attempt to deploy to production, and so on.

## The production deployment challenge
My team was fundamentally a data science team.
We were training ML models, building and running data pipelines to collect training data and train models on the latest data.
All Python code. That's all fine.

There was a big problem: the models in production were misbehaving, and customers were noticing:
> Your API returned this classifier result. That makes no sense. Why?

It would get sent to our team of analysts, and eventually my team to figure out. After a long back-and-forth, we'd conclude there's probably a deficiency in the training data, the model, or business logic wrapping the model.

> Okay, how do we fix this?
> We need to fix the model, or update the business logic wrapping the model.
> And then what?
> We can test it and deploy it to production.
> Okay, but we can't deploy to production because only engineers and ops can do that.
> What? How do they ... deploy to production?
> Go talk to them.

This became my problem to solve, perhaps heroically.

To give you a picture of where we were at, reviewing PRs was... simply not a thing.
We worked out of GitHub repositories that were largely just snapshots of the current code. That's great - it's a backup of our code.
In the ideal case, we'd push to `master`, `ssh` to an internal VM, pull the code, and run it.
In the normal case, we'd `ssh` to the VM and edit things there, rerun the models, copy the code from the VM into the GitHub repository if we remember.

How do we build a new model and deploy it to prodction? Who knows.

## The production deployment solution
This post isn't about my heroism, so to keep it brief, I leaned hard into what I understood as "DevOps":
* Went and talked to the engineering teams and the ops team. Figured out how everything fits together.
* Learned what the heck [Chef](https://www.chef.io/) is.
* Wrote and deployed an internal [PyPi repository](https://packaging.python.org/en/latest/guides/hosting-your-own-index/) that used git tags as versions, and resolved dependencies using our internal GitHub repositories. With the support of a partner in the ops team. Not a ton of code - maybe 100 lines of Python - but deeply embedded and tough to test and deploy on my own.
* Established a pattern of not just pushing to `master,` but also tagging versions to release, and sometimes reviewing code in PRs before merging to `master`
* Created a Chef recipe template for Python apps
* Created a Chef recipe for our Python app
* Deployed the thing to production, and fixed the customer's concerns!

## Since then
Since then, I've spent _most_ of my time the other side of the table, so to speak. Not to mention in a completely different company with different values, and a more flexible organization.

I've reflected a bit on what makes 2018 different from 2026.

## 2018: Production Operations Team
> We are the production operations team.
> Our mission is to protect production.

If a developer wants a change in production, it should be possible, but we need to cover our butts.
There's going to be a handoff from the developers to us. Make a full paper trail with tickets.

If production is too hard to change for a regular developer - well, that sucks, but it kind of works in our favor.
Any deviation from what's in that ticket is - by default - a liability for us.
Nobody has time for debugging a random issue that happens "only in production". We have lives outside of work!

From time to time, a hero might [push through the friction](https://blog.danielna.com/talks/pushing-through-friction) and make a self-service path. But again, that's a hero, and heroes are few and far between. They also might quickly move on to another company where change is easier.

## 2026: Platform Engineering Team
> Our mission is to accelerate development and make production resilient.

Developer experience has to be smooth.
CI/CD has to be quick.
If a developer is ever waiting for CI or CD, treat is a mini-incident.

It still feels early to be prescriptive on the _how_ and _what_, so it's worth another post in the future.
