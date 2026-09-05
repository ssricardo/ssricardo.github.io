# A Tiny OpenCode Plugin for Runtime Switches

Some controls are most useful when they can change while you are working.

An AI proxy may have a few environment-backed switches: perhaps a tool filter, memory behavior, telemetry, or verbose logging. A static configuration file is fine when you are setting up a machine. It is less pleasant when the useful question is, "can I turn this on for the next few minutes and then turn it off again?"

The `env-configs` OpenCode TUI plugin is a small answer to that problem. It places an **Env configs** section in the session sidebar and reads the proxy's current in-memory values. Each item appears as a simple Enabled or Disabled row. Click a row, and the plugin asks the local proxy to change that one value.

## A control panel, not a second configuration system

The plugin does not try to own configuration. The proxy remains the source of truth. The plugin uses `PROXIED_BASE_URL` to find it, requests the current values from `GET /env-configs`, and changes one with `PATCH /env-configs/{NAME}`. It refreshes in the background every minute and immediately updates the display after a successful toggle.

The API interaction is intentionally unremarkable. Reading the current runtime values is just a local request:

```ts
const res = await fetch(`${base}/env-configs`)
if (!res.ok) throw new Error(`GET /env-configs ${res.status}`)
return (await res.json()) as ConfigMap
```

Changing a value is equally explicit: one named setting and one new string value.

```ts
await fetch(`${base}/env-configs/${encodeURIComponent(name)}`, {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ value }),
})
```

That distinction matters. The sidebar is a lightweight view and control surface for a running process, not another file that can drift out of sync. If the proxy is unavailable, the UI says so. If a change fails, it reports the error instead of pretending the toggle worked.

## Why put this in the TUI?

Runtime choices are easier to use when they are close to the work that needs them. A developer using OpenCode does not have to leave the session, remember a curl command, or restart the proxy just to flip a temporary behavior. The visible state also makes it harder to forget that a special mode is still enabled.

This is the same general idea as per-command approval, applied at a different level. Rather than giving an AI system one fixed set of powers, keep a human-readable control nearby and make changes explicit at the time they are needed.

It is not glamorous, and that is part of the appeal. A small sidebar section, a local API, and a few clear states can be enough to make AI tooling feel less like an opaque autonomous process and more like a tool you are actively steering.
