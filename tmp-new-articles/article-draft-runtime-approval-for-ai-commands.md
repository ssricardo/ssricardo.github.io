# Let the Agent Look, but Ask Before It Touches Things

Giving an AI agent access to `kubectl` or the AWS CLI is useful right up until it is a little too useful.

The obvious answer is a hard sandbox: allow a tiny set of commands and reject everything else. That is safe, but it can also turn a capable assistant into a frustratingly polite bystander. The alternative, unrestricted command execution, has the opposite problem: a helpful request can become a production change, an expensive query, or an accidental data leak very quickly.

The small `kubectl-safe` and `aws-safe` wrappers take a middle route: safe by default, but not permanently boxed in.

## A quick first pass for routine work

The tools put a narrow allow-list in front of the real CLI. For Kubernetes, ordinary inspection commands such as `get`, `describe`, `logs`, and cluster discovery can run without interruption. For AWS, the list is made of explicitly selected read-oriented service actions, such as listing S3 buckets, describing RDS instances, or inspecting CloudWatch Logs. The wrappers keep using the developer's current credentials and context, so they do not invent another authentication system.

There are a few extra guardrails. `kubectl-safe` refuses interactive `exec` sessions because their output cannot be safely collected and filtered. It treats Kubernetes Secrets as sensitive even when the command verb itself is normally allowed, and it redacts secret data from YAML, JSON, and familiar `KEY=value` output before the agent sees it. The AWS wrapper also limits explicitly supplied regions.

The point is not to prove a command is harmless in the abstract. It is to make the common, inspect-and-understand loop easy while keeping the dangerous edge of the tool deliberately small.

The Kubernetes side is deliberately boring in the good sense: a small map documents the routine commands that may proceed.

```go
var allowedVerbs = map[string]bool{
    "get": true,
    "describe": true,
    "logs": true,
    "diff": true,
}
```

## When the agent needs an exception

An action outside the allow-list is not silently passed through. Instead, the wrapper starts a tiny web server bound to localhost, opens a browser page, and shows the requested operation. The person at the keyboard can allow it or deny it. No answer within roughly a minute means deny.

The decision point is intentionally small. If a verb is not safe, or the command is aimed at a Kubernetes Secret, the wrapper pauses before invoking the real CLI:

```go
if !verbSafe || sensitive {
    if !requestApproval(displayCmd, sensitive) {
        os.Exit(1)
    }
}
```

That makes approval a runtime decision. An agent can get as far as proposing the exact operation it needs, but it cannot make the final call for a command that crosses the normal boundary. A human can approve one specific exception without weakening the rules for every later request.

This is more practical than treating the sandbox as a wall with no door. A production investigation sometimes really does need an unusual command. The right question is usually not, "should this tool be allowed forever?" It is, "do I want *this* action, with this context, right now?"

## Not a magic security boundary

These wrappers are intentionally local helpers, not a replacement for IAM, Kubernetes RBAC, audit logs, or sound production practices. Their allow-lists and simple output filtering need to be maintained with care. In particular, read operations can still expose valuable data, and a user should read an approval prompt rather than click through it on autopilot.

Still, the pattern is pleasantly small: keep routine exploration flowing, make exceptional power visible, and preserve a human decision at the moment it matters. For AI-assisted command lines, that is often a much better fit than either blind trust or a sandbox that says no to everything interesting.
