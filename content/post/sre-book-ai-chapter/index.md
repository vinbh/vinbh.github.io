---
title: "What Google's New SRE Book Chapter Gets Right About AI in Operations"
description: "Google released two early chapters from the SRE Book 2nd Edition. The AI for SRE chapter is behind an O'Reilly paywall but a free trial gets you in. Here are the takeaways that actually matter."
date: 2026-04-29
weight: 1
categories: ["SRE", "AI", "Operations", "Reliability"]
tags: ["SRE", "AI", "LLMs", "Observability", "Incidents", "Agents", "Reliability"]
draft: false
---

Google released two early chapters from the SRE Book 2nd Edition this week. One of them is a new "AI for SRE" chapter, published on O'Reilly behind a paywall (a free trial works fine). I read it last night. Here is what stood out.

## AI is not a replacement, and the book is clear about it

The chapter does not hedge on this. Humans are still needed for high-stakes calls and for maintaining the AI systems themselves. This framing is important because a lot of the discourse around AI in operations swings between "it will replace SREs" and "it is useless hype." The chapter takes neither position. It treats AI as a capable but bounded tool that requires oversight.

That is the right framing and, frankly, the only one that survives contact with actual production systems.

## Build trust the way you would with a junior engineer

The chapter recommends staged access. Let the agent suggest fixes first. Let it handle small issues next. Expand scope only after it has demonstrated consistent judgment.

This maps almost exactly to how good engineering teams onboard new hires. You would not give a junior engineer access to run arbitrary commands on a production database on day one. The same logic applies here. The trust has to be earned through track record, not assumed because the model scored well on a benchmark.

## No rollback path means no access. Full stop.

This is the line I think most teams shipping agents today are skipping over.

The chapter is explicit: if an agent can take an action, that action must have a corresponding undo path. If there is no rollback, the access should not be granted. It is a simple rule with serious implications. A large fraction of the "AI took an irreversible action and caused an outage" stories you hear come down to this exact gap.

Most teams I see are sprinting toward autonomous remediation without auditing which of their remediation actions are actually reversible. That work is unglamorous and it is where most of the risk lives.

## Feedback loops are the same as postmortem culture

When the agent gives a wrong suggestion or takes a bad action, you flag it. The chapter draws directly on the same principles behind good postmortem culture: more context and more feedback produce better future execution.

This is not a new idea in SRE. What is new is applying it systematically to AI behavior rather than just human behavior. Teams that already have disciplined postmortem practices are going to find this easier to adopt. Teams that treat postmortems as a compliance exercise will struggle here too.

## During incidents, the value is in the searching, not the fixing

The chapter frames the agent's highest-value role during an incident as retrieval: finding the right runbook, the relevant prior incident, the ticket that describes this failure mode, faster than a human flipping between tabs at 2am.

This is accurate. I have watched skilled engineers waste 20 minutes during an active incident searching for context that existed somewhere but was not surfaced quickly. The agent does not need to push the fix to be valuable. Cutting the time-to-relevant-context by half is a significant contribution on its own.

## Dashboards tell you something is wrong. AI tells you why.

The chapter positions AI as the layer that reads tickets, user feedback, and support conversations that dashboards never capture. A dashboard can tell you error rates are up. It cannot tell you that three users filed tickets this morning describing the same broken flow.

Connecting those two signals is genuinely hard and genuinely valuable. Most observability stacks are good at metrics and logs and comparatively weak at unstructured signal from users. This is a real gap and a reasonable place to apply language models.

## Jevons paradox applied to operations

This was the framing that stuck with me most. The chapter argues that AI does not reduce SRE workload. It raises the reliability ceiling. When reliability gets cheaper to achieve, the demand for it goes up across more services. You end up doing the same amount of work but against a higher standard and across a larger surface area.

This is Jevons paradox applied to ops: efficiency gains in one area get consumed by expanded scope, not by reduced effort. Teams expecting AI to cut headcount are likely to be disappointed. Teams expecting AI to let a fixed headcount hold the line on a growing system have a more realistic picture.

## What I would add

The five-level maturity model the chapter proposes is a useful scaffold. But the gating criteria between levels is where the real engineering work lives, and the chapter is lighter on that than I would like.

"The agent suggested 50 fixes and 47 were good" sounds impressive until you ask which 3 were wrong and what they would have broken. A 94% hit rate on innocuous fixes is fine. A 94% hit rate on actions that touch shared infrastructure or modify load balancer config is a different story entirely.

Most teams I see moving toward autonomous remediation are not doing this audit. They are looking at aggregate accuracy numbers and treating them as sufficient. The chapter gestures at this problem but does not give you the tools to actually gate on it.

If you are scoping AI into your operations work in the next year, the chapter is worth a read. The framing is grounded, the concerns are real, and it avoids most of the hype you will find elsewhere on this topic.
