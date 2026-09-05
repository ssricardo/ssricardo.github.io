---
title: "Let AI Look, but Ask Before It Touches Things"
date: 2026-08-27 10:00:00 -0300
categories: [ai, security, tooling]
tags: [ai, security, tooling]
---

Giving an AI agent access to `kubectl` or the AWS CLI is useful right up until it is a little too useful. I wanted agents to help me investigate a cluster or account without making every unfamiliar command either permanently safe or completely impossible.

Hard sandboxing is the obvious answer: allow a tiny set of commands and reject everything else. That is safe, but it can turn a capable assistant into a frustratingly polite bystander. Unrestricted command execution has the opposite problem: a helpful request can become a production change, an expensive query, or an accidental data leak very quickly.

My small `kubectl-safe` and `aws-safe` wrappers take a middle route: safe by default, but not permanently boxed in.

## A quick first pass for routine work

I put a narrow allow-list in front of the real CLI. For Kubernetes, ordinary inspection commands such as `get`, `describe`, `logs`, and cluster discovery run without interruption. For AWS, the list contains explicitly selected read-oriented service actions, such as listing S3 buckets, describing RDS instances, or inspecting CloudWatch Logs. The wrappers keep using my current credentials and context, so they do not invent another authentication system.

I added a few extra guardrails. `kubectl-safe` refuses interactive `exec` sessions because their output cannot be safely collected and filtered. It treats Kubernetes Secrets as sensitive even when the command verb is normally allowed, and redacts secret data from YAML, JSON, and familiar `KEY=value` output before the agent sees it. The AWS wrapper also limits explicitly supplied regions.

The goal is not to prove a command harmless in the abstract. It is to make the common inspect-and-understand loop easy while keeping the dangerous edge deliberately small.

The Kubernetes side is deliberately boring: a small map documents the routine commands that may proceed.

```go
var allowedVerbs = map[string]bool{
    "get": true,
    "describe": true,
    "logs": true,
    "diff": true,
}
```

## When the agent needs an exception

When an agent needs an action outside the allow-list, the wrapper does not silently pass it through. It starts a small Go server bound to localhost. The server acts as a proxy to the AI Gateway: it receives the command request, pauses it, opens a browser page, then returns my decision before the real CLI runs. No answer within roughly a minute means deny.

The decision page is intentionally small: a banner shows the exact command the agent is trying to execute, followed by three choices: **Approve once**, **Approve for 5 minutes**, and **Deny**. The five-minute choice helps when I am intentionally working through a sequence of related commands, without turning that access into permanent permission.

<div style="max-width: 680px; margin: 1.5rem auto; border: 1px solid #4b5563; border-radius: 8px; overflow: hidden; font-family: system-ui, sans-serif; background: #111827; color: #f9fafb;">
  <div style="padding: 0.9rem 1.1rem; background: #7f1d1d; font-weight: 700;">AI command approval required</div>
  <div style="padding: 1.1rem;">
    <p style="margin-top: 0; color: #d1d5db;">Agent wants to execute:</p>
    <code style="display: block; padding: 0.8rem; border-radius: 4px; background: #030712; color: #e5e7eb; white-space: normal; word-break: break-word;">kubectl delete pod api-7f9d8c6b4d-x2kqz -n production</code>
    <div style="display: flex; flex-wrap: wrap; gap: 0.6rem; margin-top: 1.1rem;">
      <button type="button" style="padding: 0.55rem 0.85rem; border: 0; border-radius: 4px; background: #15803d; color: #fff; font-weight: 600;">Approve once</button>
      <button type="button" style="padding: 0.55rem 0.85rem; border: 0; border-radius: 4px; background: #1d4ed8; color: #fff; font-weight: 600;">Approve for 5 minutes</button>
      <button type="button" style="padding: 0.55rem 0.85rem; border: 0; border-radius: 4px; background: #b91c1c; color: #fff; font-weight: 600;">Deny</button>
    </div>
  </div>
</div>

*Approximation of local approval page. Buttons shown for illustration only.*

If a verb is not safe, or a command is aimed at a Kubernetes Secret, the wrapper pauses before invoking the real CLI:

```go
if !verbSafe || sensitive {
    if !requestApproval(displayCmd, sensitive) {
        os.Exit(1)
    }
}
```

That makes approval a runtime decision. An agent can get as far as proposing the exact operation it needs, but it cannot make the final call for a command that crosses the normal boundary. I can approve one specific exception without weakening the rules for every later request.

This works better for me than a sandbox wall with no door. A production investigation sometimes really does need an unusual command. The better question is not, "should this tool be allowed forever?" It is, "do I want *this* action, with this context, right now?"

## Not a magic security boundary

These wrappers are intentionally local helpers, not a replacement for IAM, Kubernetes RBAC, audit logs, or sound production practices. Their allow-lists and simple output filtering need maintenance. In particular, read operations can still expose valuable data, and I still need to read an approval prompt instead of clicking through it on autopilot.

The pattern stays pleasantly small: keep routine exploration flowing, make exceptional power visible, and preserve a human decision at the moment it matters. For AI-assisted command lines, I find it a better fit than blind trust or a sandbox that says no to everything interesting.
