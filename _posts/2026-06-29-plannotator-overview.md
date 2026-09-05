---
title: "Keep Control Over AI Plans with Plannotator"
date: 2026-06-29 10:00:00 -0300
categories: [ai, opencode, tooling]
tags: [ai, opencode, tooling]
---

When I ask an agent to handle a non-trivial task, I do not want to approve a plan quickly and discover later that it misunderstood an assumption, expanded the scope, or chose an approach I would not have picked. Reading a long plan in a terminal makes that easy to do. It is tempting to let the agent keep going when what I really need is a pause to understand and shape the work before files change.

[Plannotator](https://github.com/backnotprop/plannotator) gives me that pause. It is a plugin for OpenCode that opens a richer interface for reviewing an AI-generated plan. Instead of treating planning as a formality before implementation, I use it as an early review point: I read what the agent intends to do, question parts that do not match my intent, and ask for a revised plan before approving any implementation.

## How I Use It

Plannotator is inserted into OpenCode's `submit_plan` phase, so I start the task in plan mode. Once the agent has analyzed the request and prepared its approach, the plugin presents that plan for review instead of letting implementation begin immediately.

My usual loop is:

1. Start a task in plan mode and let the agent investigate the problem.
2. Read the proposed implementation before code changes begin.
3. Add annotations to specific steps when an assumption, scope boundary, or technical direction needs adjustment.
4. Leave a general comment when the whole approach needs to change.
5. Review the revised plan, then approve it when I understand and agree with the intended work.

That is more useful to me than a generic "looks good" response. A comment attached to an exact step gives the agent a clear correction, and it makes it easier for me to see what it plans to change. More importantly, it keeps me involved while the work is still cheap to redirect. I can understand the proposed implementation and make adjustments early rather than letting the agent get far into the task before finding a mismatch.

![Plannotator plan review interface]({{ site.url }}/images/articles/plannotator.png)

*Reviewing and annotating an agent plan before implementation.*

## Why It Has Stuck

The value is not that every task needs a formal review. Small, well-understood changes usually do not. But when a task touches several files, has unclear requirements, or could lead the agent down an expensive path, I prefer taking a few minutes to review the plan first.

For those cases, Plannotator gives me a better checkpoint between asking for work and letting an agent do it. I get a clearer view of what is going on, can correct the direction while it is still a plan, and retain control over how much autonomy I give the agent.
