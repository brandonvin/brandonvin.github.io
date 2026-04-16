Migrations considered helpful! And the tale of the Matryoshka Migration
===

![Seven Matryoshki](/img/matryoshki-ye-olde-curiosity-shop.png)

_Seven Matryoshki. Source: Ye Olde Curiosity Shop, Seattle, WA_

Migrations are an art, not a science. A migration is a juggling act between
breaking glass, keeping the lights on, and opening new doors.

Migrations are the evolution of a system through incremental changes.

Migrations are not limited to infrastructure; customer/product changes can be
migrations too.

The difference between a single commit and a migration is that a migration is a
long series of commits. Also, migrations generally affect a lot more than one
person.

Migrations require updating docs, people's brains, and muscle memory.

Done well, migrations reduce complexity and accelerate product development.

Done poorly, migrations increase complexity and slow product development.


# Why did I write this?

Circa November 2024, some seasoned engineers, a few newer folks, and I, had a passionate discussion:

> We’ve increasingly moved our product towards being **easy** -- wizards, fast
> onboarding -- at the cost of **simplicity** -- removing non-essential
> complexity. Does that resonate with anyone else? The easy things become quite
> hard to manage in the medium-to-long term.

From what I heard, yes. It resonated, resoundingly.

> How do we think about the tradeoff between **simple** and **easy** in a longer view? If we
> build something easy today, how do we think about incremental architectural
> changes to reduce our complexity? Likely easier said than done, but how have we
> thought about it in the past, vs. now?
>
> What are the biggest sources of pain and complexity right now in the platform,
> and are there ways to reduce that complexity over time, balancing it with
> product roadmap, new features, etc.?

These are great questions!

This conversation happened in November 2024, when I had just completed a 3-year migration project
of my own. So I was already in a headspace of reflecting on the past 5 years and what wisdom
can be gleaned from it.


# So what's the goal?

At Amperity, the word "migration" carries a lot of historical baggage.
Let's sit with the baggage, process it, and accept it.

Let's:

* Share examples of good outcomes of migrations.
* Share examples of troubled migrations and how they turned successful.
* Share common themes from the above.


# How do big migrations start?

Usually, but not always, a design-and-architecture or staff engineering group
gets together and studies a big problem. A design doc, or series of design docs
get written. It may be very long, 20+ pages. Usually, an engineer is chosen as
the Directly Responsible Individual (DRI) to start and navigate the project.


# Examples of successful big migrations

We did it! Yay, us! This is in no particular order, and I'm obviously forgetting some as I write this:

| Before | After | Big unlocks (non-exhaustive) | Est. finish date |
| :---- | :---- | :---- | :---- |
| Workflow System V0, built on Apache Airflow | Workflow System V1 | Customers can own Workflows -- Reliable Workflows -- Customer vs. Platform Attribution of Failures | 2024 |
| Finagle/Thrift Service Framework | Prodigal Service Framework | Developer Efficiency | 2022 |
| Self-Running Apache Kafka on VMs in the cloud | Purchase Confluent Kafka-as-a-Service Kafka Topics Managed in Infrastructure as Code in a CD pipeline | Developer Efficiency -- Infra Cost Efficiency | 2022 |
| Cloud Storage Spaghetti | Storage Service and APIs | Bridge / Delta Sharing -- Strong security posture -- Bring Your Own Storage | 2024 |
| Jobs Inputs/Outputs in Cloud Storage | Job Inputs/Outputs in Redis | Developer Efficiency | 2022 |
| API Framework V0 | API Framework V1 | Developer Efficiency | 2022 |
| ADLS Gen 1 | ADLS Gen 2 | Infra Cost Efficiency -- Platform Reliability -- Security Posture -- Developer Efficiency | 2021 |
| Loggly for structured logs \+ Honeycomb for traces | Honeycomb for traces and structured logs | Infra Cost Efficiency -- Developer Efficiency -- Security Posture | 2023 |
| Plain Text Logging | Structured Logging and Tracing | Speed to debug/resolve incidents -- Developer Efficiency | 2021 |
| Deprecated Azure Public IP | Modern Azure Public IP | BFD: Connections to customer systems won't break when Azure retires their old infra -- Developer Efficiency | 2024 |
| Loading Dock | Not Loading Dock | Security Developer Efficiency | 2024 |
| Tables In Accumulo \+ HDFS | Files in Cloud Storage, removal of Accumulo/HDFS | Infra Cost Efficiency -- Developer Efficiency | 2020 |
| HDFS | Cloud Storage, full removal of HDFS | Infra Cost Efficiency -- Developer Efficiency | idk |
| Product Configuration in Spaghetti/Accumulo | Unified Product Configuration | Sandboxes -- Developer Efficiency | idk |
| Service Data in Spaghetti/Accumulo | Service Data in PostgreSQL | Sandboxes -- Developer Efficiency | idk |
| Something I'm Forgetting | Something I'm Forgetting |  | idk |

**TODO:**  
I've worked on backend/infra most of my time at Amperity, so I'm probably missing some of the more direct customer-facing migrations. Feel free to comment here\!


# What's in a successful big migration?

Usually:

* A **Directly-Responsible Individual (DRI)** who is also a direct contributor to the code (or holds enough sway with engineers who *are* direct contributors)  
* A central **Design Doc,** kept up to date. A Google Doc works well.  
* Maintain **buy-in** from engineering and product  
  * You will be asked, "What?" and "Why?". **Do Repeat Yourself**, and link to the **Design Doc**.  
* **Small focused team**: Team-to-team handoffs and trying to coordinate across teams is the path of pain.  
  * This may include **embedding**: bringing together SMEs from different teams (e.g. a product domain SME with a cloud infrastructure SME) into a "tiger team" or "v(irtual) team"  
  * Observe the Churn Rule: take the initiative on updating callers, in backward-compatible fashion. Avoid imposing work on teams who are likely busy and far from the migration details  
    * Friction in the migration strategy directly feeds back into accelerating the migration (e.g. they may start with a runbook, then feel the repetitive parts, and then automate them)  
* **Ratcheting:** when the new pattern is ready and in use, prevent new usages of the old pattern from being added \- usages of the old pattern monotonically decrease.  
  * See also: [Strangler Fig Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)  
  * See also: [Shitlists](https://sirupsen.com/shitlists) (CI tests that fail when old pattern is re-introduced)  
  * Examples at Amperity:  
    * `mulch` is a homegrown Shitlist for Clojure code. It blocks new usages of deprecated functions in our Clojure and ClojureScript code.
    * After we migrated a few services from Finagle/Thrift to Prodigal, we produced a runbook to migrate services and made Prodigal the officially blessed path for new services.  
    * After we migrated a few services from Aurora to K8s, we produced a repeatable migration runbook. We updated the "add a new service" tools/docs/guidance to target K8s. Soon after, 5 new services needed to be added, and they were stood up directly on K8s.  
* They are finished and **recognized as finished**. How?  
  * Written and shared definition of done  
  * Announcement that it's done  
* Deliver an early win and/or other incremental wins  
  * Caveat: Avoid starting with the easiest parts  
    * Up front research to find the riskiest, hardest parts and derisk them  
    * Sequence work so that the hard parts get easier  
      * Example: we don't know how to use K8s, and have no K8s tooling, which is a big operational risk we need to solve before using it widely in production → Let's start by using K8s in a lesser-used part of the product to build expertise and foundational tooling  
    * Or, start with the riskiest part to prove the migration strategy  
      * Example: in both the Prodigal and K8s migration, one of the first services we tackled was Tenant Service, which is a high-traffic, core service that could bring the whole app down  
    * Sometimes migrations stall because they dive head-first into the easy parts, and then the harder parts lead to re-considering earlier design decisions.  
  * **Also, Scenic Routes**: take a little detour to get obvious wins for customers, which are clearly beneficial to the migration  
    * WARNING: when Scenic Routes keep getting pulled into the critical path, and they grow in scope, this can lead to a Matryoshka Migration  
* Learn from Incidents  
  * We run AARs and incorporate the learnings into the **Design Doc**  
* Delete the dead code\!  
  * If the only dead stuff is code, it's often easiest to delete it all at once, when we're confident it's unused. Big red line PRs are tasty\!  
* Shut down the dead infra\!  
  * In infra migrations targeting cost efficiency (e.g. shutdown of Accumulo/HDFS), slowly scaling down the old system helps deliver incremental wins.

## How do big migrations struggle?

### Pattern: Matryoshka Migration

Commonly, migrations turn into **Matryoshka Migrations**. [Matryoshka](https://en.wikipedia.org/wiki/Matryoshka_doll) is the Russian nesting doll. 🪆Migrations nested in migrations.

A Matryoshka Migration is one that has diverged a bit too far from its original goals.

This migration has gone on so long that we started other migrations that affect the same parts of the system.

The critical path of the migration has gotten longer and more complicated, rather than shorter and simpler.

You can smell it:

* A migration started  
* It has been *almost done* for a really long time: months, or years.  
* Though it's done with good intentions (e.g. **Scenic Routes** motivated by product/customers), the migration keeps going… and going… and going… or pausing inexplicably.

### Pattern: Drained DRI

For example, a migration was started 1 year ago. Then, re-orgs and staffing changes (e.g. hiring, attrition, reduction in force) happened.  
All the teams, code ownership, and managers have shuffled around.  
Company strategy can change, too.

However, the friction that motivated the migration, and the friction of unfinished migrations, usually rear their ugly head again.  
It's hard work for the DRI to re-establish buy-in and kickstart the migration. Sometimes they have to do this multiple times.

What can help:

* Design Doc  
* Switch DRI  
* Embedding, re-establishing a focus team  
* Let the DRI take a break

## Turn this Matryoshka around\!

Some of the successful migrations became Matryoshka Migrations and then turned around. How?

* Recognize that it's a Matryoshka Migration.  
  * Compare the original goals/timeline with the current timeline. A smell is that the original timeline hasn't been re-evaluated.  
* Re-evaluate the cost of complexity inherent in having multiple, large in-progress migrations.  
* Re-focus on the original goals. Telling the customer value, and buy-in of those goals.  
* Re-evaluate what's really in the critical path, and focus on it.  
* Re-establish a small focused team with a "mandate", free up their time to focus on it.

# Own up to blocked and stalled big migrations

Let's be leaders: communicate, and have empathy.

Yes, it is **okay**, and sometimes **better for the company** to wait to start, pause, or stop a migration.

There's a negative outcome we've seen sometimes at Amperity. Example:

1. Engineers or customers feel pain from a problem that exists **today**  
2. Engineers have designed a solution, socialized it with engineers, and feel that they have clear buy-in.  
3. Engineers feel blocked from starting or continuing work that they believe would solve the problem.  
4. This blockage lacks a clear explanation: nameless leaders withhold their buy-in  
5. **Outcome:** a tenured, high-value engineer is demoralized.

**If we decide to block, pause, or cancel a migration, let's communicate clearly and honestly why.**

# Reflect on finished big migrations

When a migration finishes, we can run retrospectives to gather and share what we learned. **Consider this doc a "Meta-Retrospective" of 5 years of migrations at Amperity.**

One example retrospective is the `Spark 2-to-3 Retrospective`.

# So… what should we *do*?

It's a fact. We have a complex product and complex infrastructure.  
What are the biggest sources of complexity right now?  
Are there ways to reduce that complexity over time, balancing it with product roadmap, new features, etc.?

Yes\!

We'll do all the migrations, all at the same time, all as fast as possible. No \- sorry, that's a joke. **Today, we can't do all of them at full speed.**

* As engineering leaders (e.g. staff-eng community), let's catalog and **prioritize** all our major migrations that are executing and proposed.  
* If some migrations are entering Matryoshka status, and still valuable, let's focus on finishing them.  
* We should decide which ones to pause, and which ones to hold off on starting, with **empathy**.  
  * Note that pausing may mean some progress may still happen \- it just will be in the background with gradual progress, likely not on any team's Jira board. Perhaps interspersed among new feature commits, written on quiet Wednesday and Friday afternoons. (See below: Tidy as we go)

We’ve done a lot of the hard thinking on ways to simplify that complexity, but we haven’t started or implemented them fully.

Examples:

* Datasets: groups of named tables. Tables are tables!
* Amperity's internal library, `comp-graph` for forming Computation Graphs of complex layered transformations. (Credit to Kevin Litwack)
* Amperity Sandboxes: a massive enabler for testing, sharing changes. I think this is done from a product standpoint -- what is left from the engineering perspective?
* Coordinated Changes
* Explicit vs. implicit configuration / semantic action-at-a-distance  
* Workbench for exploring Datasets (state)  
* Drafts

## Tidy as we go

We can pragmatically "Tidy as we go". [Tidy First?](https://www.oreilly.com/library/view/tidy-first/9781098151232/)  is a nice 2-hour read on this topic.

Quick summary:

* "Tidying" is at the level of individual commits, a conversation among engineers and the code; it's not called out as a "migration project".  
* Tidying means effectively carrying out a "low-urgency migration" bit-by-bit, interspersed with new-feature commits. In other words, gradual refactoring towards a higher quality, lower-friction codebase.  
* Tidying requires trust and good judgment.

Example at Amperity:

* UI Code Modernization: Reframe Spaghetti → React/Helix


# Sources and inspiration

This doc is a synthesis of my own experience and several other people's great ideas.  

```
(iterate inc @amperity-engineering)
```

I've chatted about this topic with these people at Amperity *(please note, I took 5 seconds on this list, it is not exhaustive, and in no particular order)*:

  * ... many others!
  * John Rush  
  * Jeff Stokes  
  * Greg Look  
  * Stephen Meyles  
  * Hemanth Srinivas  
  * Aria Haghighi  
  * Bryce Covert  
  * Brandon Vincent  
  * Ace Levenberg  
  * Drew Inglis  
  * Graeme Roche  
  * Kevin Litwack  
  * Cary Lee  
  * Joe Christianson  
  * ... many others!

In general, thanks to Amperity's `#staff-engineering` discussions.


... and in the public domain:

* [Kent Beck -- "Tidy First?"](https://www.oreilly.com/library/view/tidy-first/9781098151232/)  
* [Dan Na -- "Pushing Through Friction"](https://blog.danielna.com/talks/pushing-through-friction/)  
  * What is **friction**? It is resistance; when things feel harder than they ought to be.  
  * **Friction** in engineering demotivates otherwise smart and highly-motivated engineers.  
  * **Friction** in a product turns customers away.  
  * "**WTF Factor**" and "**Normalization of Deviance**": The harms when tech debt, stalled migrations, or struggling migrations "swept under the rug" become normalized in an organization.  
  * "**Pushing Through Friction Is The Job** *\[of a tech lead or staff+ engineer\].*"  
* [Tanya Reilly -- "The Staff Engineer's Path"](https://www.oreilly.com/library/view/the-staff-engineers/9781098118723/)  
  * Chapter: Leading big projects  
  * Section: Why have we stopped?  
* Will Larson's astute articles  
  * [Migrations: the sole scalable fix to tech debt.](https://lethain.com/migrations/)  
  * [Your migration probably isn’t failing due to insufficient staffing.](https://lethain.com/migration-isnt-failing-due-to-lack-of-staffing/)  
* [Matt Schellhas -- "You will always have more problems than engineers"](https://betterprogramming.pub/you-will-always-have-more-problems-than-engineers-aafff94a4623)
* While we were pretty proud of "coining" the term Matryoshka Migration, it
  turns out we aren't really that original. Except when it comes to naming
  things. Check out this travelling blog: [Migrating Matryoshka](https://migratingmatryoshka.com/)
