---
title: "Revitalizing a Legacy Project: Empower an Overwhelmed Team."
seoTitle: "Empowering Teams to Revitalize Legacy Projects"
seoDescription: "Learn how a legacy software project was rescued from chaos. Discover actionable steps to reduce overload, rebuild trust, and empower teams."
datePublished: Thu Apr 03 2025 21:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cmaezj0tz000k09jobwhj31l9
slug: empower-an-overwhelmed-team
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1746511355980/fd955958-267b-4924-9f67-4f3163d271e7.png
tags: productivity, kanban, software-engineering, developer-experience, legacy-systems, engineeringleadership

---

Imagine inheriting a 25-year-old project weighed down by technical debt. The team is exhausted from constant firefighting, while stakeholders are dissatisfied with the project’s outcome and label it "stuck." Years of mergers and acquisitions resulted in the loss of key domain experts and significant changes in product requirements, which the team struggled to manage, leaving the project in chaos.

However, we turned things around. It took a couple of years, but today, instead of being seen as “stuck,” we’re often recognized as one of the most efficient teams in the company. And the journey to get here taught us a lot.

In [the previous article](https://ordinarytech.blog/overcoming-teammates-overload), I shared how to help overloaded team members. Unsurprisingly, this wasn’t just a few people under stress — the entire team was under pressure. In this piece, I’ll focus on how we addressed that team-wide overload.

When I joined, I kept hearing the same thing from teammates: “Stakeholders don’t let us address technical debt. They keep overloading us with more and more requests.”

So were the stakeholders the villains? I don’t think so. The real issue was that **stakeholders didn’t understand the team’s actual capacity**. And to be fair, **how could they if the team didn’t either**?

Once we started paying attention, we learned a lot about our own work and how to make it more efficient, predictable, and focused. Let’s start with uncovering the gap between expectations and what the team could realistically deliver.

# What Was the Team Expected to Deliver?

At the time I joined, there wasn’t much clarity around how work was structured or what the team’s actual capacity was. That’s not unusual. When you’re constantly putting out fires, it’s hard to find time to zoom out and look at the bigger picture.

So I spent my first three months just observing how the team worked and what stakeholders typically expected from us. There were six engineers, and here’s how their work was *supposed* to be split:

[![A chart with three sections labeled "Epics," "Standard," and "Support+Intangible." Each section has two or three items, each prefixed with a person emoji, with labels like "Epic 1," "Standard 1," and "Support ticket 1."](https://cdn.hashnode.com/res/hashnode/image/upload/v1746089957106/276f78b1-5d64-4eab-a71e-8cac2a1a7f6e.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746089957106/276f78b1-5d64-4eab-a71e-8cac2a1a7f6e.png)

<center><small>Initial expectations from the team</small></center>

* **Epics** were large, high-priority initiatives. They were usually broken into multiple tasks, often involved other teams, and came with deadlines. Quality was expected from day one. My team was expected to handle three Epics at the same time.
    
* **Standards** were smaller feature requests. They typically didn’t have hard deadlines, but the team had adopted a Kanban-style SLA: Standard tasks shouldn’t sit in the backlog for more than six weeks. In theory, SLAs can be helpful. In our case, though, it backfired — this rule blurred the team’s focus, and team ended up spending time on a lot of low-value work just to meet the SLA.
    
* **Support** included customer issues that frontline support couldn’t solve and ad hoc requests from other teams using our system. The expectation here was a quick turnaround — basically, “fix it now”.
    
* **Intangible work** was another [class of service borrowed from Kanban](https://www.scrum.org/resources/blog/classes-service-kanban-what-are-they). For us, it meant technical debt. One engineer was supposed to handle it, but only if there were no Support tickets waiting which, of course, happened rarely.
    

The first clear problem with this setup was that **every task came with some kind of commitment** — a deadline, an SLA, or the urgency of a support ticket. In practice, this meant each engineer was always tied up. So when someone got stuck or an incident happened (which was often), the only option was to drop some commitment. We had no flexibility to absorb surprises, and since surprises were routine, we routinely failed to deliver on our promises. To put it plainly, **our initial setup made failing commitments almost inevitable**.

**The second issue was siloed work.** Each engineer had their own tasks and mostly worked alone, which meant limited knowledge sharing. Since we were always under pressure, tasks were assigned to those who already had the necessary expertise, leading to even more knowledge silos over time. This, in turn, led to multitasking because it was rare for the most knowledgeable person to be available when an urgent request came up.

The third problem was that **all of this was considered the baseline**. So whenever someone took a vacation, got sick, or left the team, something would fall behind. If you do the math — six engineers, each with four weeks of vacation — almost half the year, someone is unavailable. Yet, the expectations never changed.

These are classic signs of an [overutilized team](https://longform.asmartbear.com/utilization/), and that was considered a minimal expectation for us.

But the reality was even worse because those four categories didn't cover everything.

# Facing Reality: The Unseen Workload

While clarifying expectations, I also tracked what was actually happening in the team's daily work. It didn't take long for the numbers to reveal their own story: on average, we dealt with **five to six urgent tasks per month**, whether it was a production incident or a last-minute business request.

For example, our Kafka mirroring cluster went down three times during my first two weeks on the project. Each time, the team hurried to get it running again. The better solution was to move a couple of services to the other data center and get rid of this mirroring cluster, but that required more effort. When you're always in emergency mode, there's no time for strategic fixes.

According to the data I gathered, such tasks took an average of four days to resolve, meaning that **the team was dealing with at least one unexpected, urgent issue at any given time.** So, the reality was that we had an ongoing stream of unseen work that wasn't considered in the expectations, yet it took up a significant amount of our time and energy.

[![The same tasks board with additional category of Expedites. Expedites highlighted with a note saying "6 expedites per month according to stats."](https://cdn.hashnode.com/res/hashnode/image/upload/v1746259853405/a5aa84c7-db69-41c3-8c46-9ecf17abac7f.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746259853405/a5aa84c7-db69-41c3-8c46-9ecf17abac7f.png)

<center><small>The reality: we have 6 Expedites monthly and no planned capacity to address them</small></center>

# Aligning Expectations with Reality

Given the lack of trust from stakeholders, I knew we couldn’t overhaul everything overnight. We needed to take small, deliberate steps.

## Step 1: Minimizing Distractions

The first priority was to **carve out space for the team to focus** by reducing the constant stream of expedites.

I shared my findings with the VP of Engineering using the same images you saw above. To address the issue, I proposed reducing the number of simultaneous Epics from three to two and introducing a weekly rotating **Quarterback** role within the team.

**The Quarterback would** **handle anything that threatened the team’s progress**, whether it was an urgent incident, a teammate stuck on a task, or an unexpected blocker. Essentially, the Quarterback’s role was to manage chaos so the rest of the team could remain focused.

In the short term, this should help us to shield the team from distractions. In the long term, the goal was to get ahead of recurring problems and fix them at the root so we’d stop getting pulled into emergency mode so often.

[![The same tasks board with added "Quarterback" emoji that is connected with arrows with each of the swimlanes meaning that this roile is intended to work on any of the swimlanes.](https://cdn.hashnode.com/res/hashnode/image/upload/v1746344060834/7f4bebb3-70cb-4421-bafb-90abf6e6ced1.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746344060834/7f4bebb3-70cb-4421-bafb-90abf6e6ced1.png)

<center><small>Quarterback "attacks" everything unexpected, keeping the rest of the team focused.</small></center>

We also started tracking where the Quarterback spent their time each week.

At first, the quarterback role was fully occupied with the “red swimlane” — urgent issues and constant firefighting. But instead of just applying quick fixes, we started tackling the root causes behind these incidents. That shift to a systemic approach paid off. We **reduced the number of monthly incidents from six to one** (and, as of 2025, we have one incident per quarter). In addition, we’re now resolving incidents much faster, thanks to having a dedicated teammate to solve them and to improve our most painful troubleshooting tools.

Eventually, the Quarterback role began shifting toward less urgent "yellow swimlane" tasks, indicating we were ready for the next step.

## Step 2: Shifting from Risk Reaction to Risk Prevention

The next thing to tackle was our approach to technical debt.

Stakeholders often complained that the team spent too much time on “technical stuff” instead of delivering visible progress. At the same time, engineers were frustrated that they *couldn’t* spend time on technical debt at all.

As usual, the truth was somewhere in the middle.

Officially, one engineer was supposed to handle technical debt (tasks labeled as “Intangible”), but only if there were no Support tickets in the queue. **According to the data, we averaged four Support tickets at any given time.** **As a result, technical debt was never addressed in a planned manner.**

This wasn't new. A couple of years earlier, leadership decided to delay addressing the technical debt "until things got better." Of course, things didn't improve; they got worse. The team still dealt with tech debt, but only when it blew up as production issues — another instance of unmet expectations, leaving stakeholders frustrated. So, we *actually* *were* spending time on technical issues, but only *reactively*, when they’d already caused damage.

One example really highlighted this issue: we had a backlog item to upgrade CentOS 6, which reached its end-of-life in November 2020. We knew about this issue, but never had time to prioritize it. When November 30 arrived, CentOS 6 was removed from official repositories, and our delivery pipelines broke.

We got the pipelines working again with a custom image. But here was the bigger issue: running production on an unsupported OS lowered our company’s security rating, hurting its reputation and raising insurance costs.

So we had to drop everything and urgently upgrade the OS. Except it wasn’t just a simple upgrade — some of our customers were still using outdated SSL/TLS protocols that newer OS versions didn’t support. That meant we had to coordinate changes with customers before proceeding. It turned into a months-long distraction, all because we hadn’t addressed a well-known risk in a planned way.

This made one thing painfully clear: the so-called “intangible” work had very *tangible* consequences.

I went back to our VP of Engineering with a new proposal. We would remove the unclear "Intangible" label and stop using the overused term "technical debt." Instead, we would **divide technical work into two clear categories: risk reaction and risk prevention**. We would allocate planned capacity to focus on risk prevention, encouraging the Quarterback to spend as much time on this area as possible.

The goal was simple: tackle potential issues before they became emergencies.

[![The same tasks board, but the numer of Expedites reduced to 1 per month from 6. New swimlane named "Risks prevention" added.](https://cdn.hashnode.com/res/hashnode/image/upload/v1746261633929/8b6fae1c-033a-4048-9c17-f32d28b1578d.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746261633929/8b6fae1c-033a-4048-9c17-f32d28b1578d.png)

<center><small>The number of Expedites has been reduced to one per month. Now we are able to proactively eliminate technical risks.</small></center>

## Step 3: Empowering the Team with Autonomy

Once we reduced the number of emergencies, the team finally had room to breathe. We could meet our commitments more consistently and, just as importantly, start rebuilding trust with stakeholders.

The Quarterback began spending more and more time in the risk prevention swimlane. They addressed long-standing technical issues, resolved major security problems, improved diagnostics and support tools, and fixed weak areas in our delivery pipeline. These changes made the entire team more productive and helped rebuild stakeholders' trust even more.

With that gained trust, we made three more important changes:

1. **We got rid of the six-week SLA for Standard tasks.** It had been pushing the team toward overcommitment and inflexible planning. After a lot of discussion, we agreed this practice was doing more harm than good.
    
2. **We stopped requiring approval for each individual technical task.** At this point, we were consistently delivering what stakeholders expected, which gave us enough credibility to take more ownership. We no longer had to negotiate each technical task. Instead, we began focusing on what was most valuable at the moment, with an agreed-upon capacity of one teammate, rather than sticking to decisions made weeks earlier.
    
3. **We shifted from the initial “push” approach** — assigning tasks to the team based on a monthly plan — **to a “pull” model**, where each teammate picks up the next task only after completing their current one. If someone is blocked, they don't take on a new task. Instead, the team focuses on resolving the blocker. If the blocker will be present for a long time and can't be addressed immediately, a teammate first offers help to others with their tasks before taking on a new one.
    

By now, most of the technical risks had been addressed, which meant we could shift more energy toward planned *development* work.

And since this was a 25-year-old system that had been under constant pressure for years, there was no shortage of messy problems that needed fixing. One example: our end-to-end tests were running on a version of Chrome that was 40 versions behind the current one. Obviously, those tests couldn’t reliably tell us if the app worked for real users.

Now we've got both the time and the autonomy to clean up that kind of stuff, too.

[![The same tasks board, but there is no more Expedites swimlane and Quarterback emoji. "Risks prevention" swimlane is renamed to "Technical development". Each of the swimlanes has at least one dedicated teammate.](https://cdn.hashnode.com/res/hashnode/image/upload/v1746262473121/604fad2d-2527-4d30-8d1c-df2b72cea8bf.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746262473121/604fad2d-2527-4d30-8d1c-df2b72cea8bf.png)

<center><small>The Quarterback role is no longer needed. We can plan our work and be confident that the plan is realistic.</small></center>

## Step 4: Achieving More with Less

The changes we’d made so far helped a lot. We improved our technical foundation, reduced emergencies, and started delivering more predictably. But some deeper issues remained.

First, **everyone was still working solo**. That limited knowledge sharing and made the team fragile — if someone was out sick or on vacation, progress stalled. Also, some of our Epics could’ve moved faster if we had worked together, but we weren’t set up for that.

Second, we had already dealt with the easy parts of technical debt. What remained were large, high-impact projects, like moving all secrets (passwords, tokens, certificates) to the [Vault](https://www.hashicorp.com/en/products/vault) to secure our delivery chain and improve our audit readiness. **But these efforts required months of focused, coordinated work — something our current setup just wasn’t built to support.**

So I went back to the VP of Engineering and pitched a new approach:

[![The same tasks board but with only two swimlanes left: "Epics" and "Standards". "Epics" has four people on it, "Standards" swimlane has two people. ](https://cdn.hashnode.com/res/hashnode/image/upload/v1746093086280/0857d2e0-70f6-49a2-aa3b-d3dbd69f9dd5.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746093086280/0857d2e0-70f6-49a2-aa3b-d3dbd69f9dd5.png)

<center><small>Now work is completed within small teams, knowledge is shared, and no one works alone.</small></center>

**1\. Fewer activities but approached as teams**

We collapsed our work into two swimlanes: **Epics** and **Standards**, not based on priority, but on *how* the work should be done.

* **Epics** are major initiatives that require focus, flexibility, and collaboration. Success means delivering on time and with high quality. So, we assigned four of our six engineers to this swimlane and let them self-organize. They could choose to work as one team of four or split into two teams of two. They could decide whether to pair-program or mob-program to share knowledge or work on tasks in parallel. We kept a hard limit of two concurrent Epics to prevent overstretching.
    
* **Standards** became a catch-all for smaller work: feature requests, minor tech tasks, and support tickets. Two engineers handled this swimlane and had full autonomy to decide how to balance the workload. If Support tickets were piling up, they could team up to get through them faster. If things were quiet, they could knock out Standard tasks.
    

**2\. "Standards" is an adjustable swimlane**

If the Standards swimlane wasn’t busy, we had the flexibility to temporarily move one or both engineers to help with a certain Epic. This gave us the agility to handle high-impact work in the best possible way.

**3\. Rotations for growth and resilience**

To avoid burnout and build shared knowledge, we introduced regular rotation between the two swimlanes. Everyone got the chance to work on both Epics and Standards swimlanes. This encouraged skill growth, avoided silos, and promoted natural knowledge sharing through pair or even mob programming.

The biggest benefit of these three changes was that they gave us the **autonomy to decide how to do our work**. Of course, autonomy requires a certain level of team maturity. But the thoughtful steps we’d taken over the past two years helped us get there. Each team member now understands the value of our work, how we make and keep commitments, and how we handle risk.

Another significant change was that **we started doing fewer tasks, but with more focus**. We reduced our commitments from six (sometimes more) to just four. This allowed us to complete tasks faster and with higher quality. It also made the team more resilient: if someone working on an Epic went on vacation, progress didn’t stop. Planning time off became easier and less stressful.

This is how “doing more with less” looks in action: fewer things in progress, but more things getting done.

And it worked.

We started consistently meeting — and often exceeding — business expectations. The team gained a reputation across the company as one of the most effective, reliable, and forward-thinking. We became a go-to example of how to build mature development practices: [integrating security into development](https://ordinarytech.blog/integrating-security-into-development-process), [adopting continuous delivery](https://ordinarytech.blog/dora-metrics), and systematically eliminating whole classes of risk.

# What Was the Impact of These Changes?

I believe the most meaningful measure of success is the change in sentiment. Upper management no longer labels us a “stuck” team—in fact, we’re now often cited as the most efficient team in the company. Burnout has noticeably declined, and team members report feeling far more satisfied with their work. They now see their contributions as impactful and meaningful.

However, I intended to give you some numbers.

On average, we now complete tasks **five times faster than before**. Reducing multitasking, enabling deep focus, sharing knowledge, improving the codebase, and investing in tooling helped us speed up every phase of delivery: development, code review, testing, and deployment.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1746603904615/b102be3d-ea4f-435d-854a-379d71566f81.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746603904615/b102be3d-ea4f-435d-854a-379d71566f81.png)

<center><small>Average Task Cycle Time (from implementation to deployment): Summer 2023 vs. Summer 2020</small></center>

Even more importantly, our **predictability** has improved. The standard deviation of task completion time has **dropped 4×**, thanks to reduced uncertainty in the codebase and tooling, more pairing, and fewer interruptions. Today, when we commit to something, we do it with much more confidence.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1746604097599/94101326-968e-4cb3-9e80-c3c491278099.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746604097599/94101326-968e-4cb3-9e80-c3c491278099.png)

<center><small>Task Cycle Time – Standard Deviation: Summer 2023 vs. Summer 2020</small></center>

Perhaps the most illustrative metric is our **average pull request lead time** — the time from the first commit to merging into the main branch. It captures the full story: the dip in performance after the company split and subsequent acquisition, the impact of losing key engineers, and then the steady improvement after we reshaped how we work. When I joined, PR lead time averaged 17 days. Today, it’s just 1 day:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1746606067483/7146ac65-ca02-4ea3-83e9-f763b92c3dd9.jpeg align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1746606067483/7146ac65-ca02-4ea3-83e9-f763b92c3dd9.jpeg)

<center><small>Pull Request Lead Time – From First Commit to Merge into “main”</small></center>

Of course, that single metric doesn’t capture everything, but it shows how much is possible when you stop overwhelming a team and give them the space to focus.

It’s difficult — if not impossible — to isolate the impact of any single change. The improvements we’ve seen are the result of many adjustments compounding over time. For example, [integrating security into development](https://ordinarytech.blog/integrating-security-into-development-process) and [achieving measurable delivery improvements with DORA metrics](https://ordinarytech.blog/dora-metrics) (among others, I haven’t written about yet) significantly boosted our effectiveness, and none of it would’ve been possible without the process changes described in this article.

Are you curious about the other changes we introduced to make this happen? That’s exactly why I created this blog. I’ll be sharing more stories and lessons — stay tuned.

# Conclusion

When a team struggles to meet expectations, the reflex is to tighten control. But in reality, the opposite is often needed: **teams need some slack to absorb risks**.

What I hope this story shows is that meaningful change doesn’t have to start big. You can shift a team out of firefighting and rebuild trust through small, deliberate steps: aligning expectations, proactively resolving tech debt, letting the team shape how they work, and doing fewer things better.

Much of what worked for us reflects core Kanban practices: start with what you have, limit work in progress, make process policies explicit, and let the team self-manage and focus on outcomes.

Is our current approach perfect? Not at all. Swimlane rotations rarely line up cleanly — people finish work at different times, and deep-focus tasks don’t pause just because it’s rotation day. Moreover, Support and Standard tracks often drift apart again, with one person handling all the Support requests while another works through Standard tasks alone.

But that’s okay — this isn’t a fixed configuration. Continuous improvement is about learning, adapting, and making the system a little better. Repeatedly.

---

Life is so beautiful,

Fedor