# The Shortcut Is the Signal: Introducing Friction Intelligence

*Why the biggest risk in your cluster usually isn't a sophisticated attack — it's the workaround someone reached for because the platform made the right way harder than the risky one.*

Most Kubernetes security incidents don't start with an attacker. They start with an operator who was just trying to ship something, hit a wall the platform put in front of them, and found a way around it.

## The problem

A wildcard RBAC role, granted because the ticket for scoped access takes two weeks. A shared kubeconfig, passed around because SSO federation broke last quarter and nobody fixed it. A `kubectl exec` straight into a production pod, because the deploy pipeline has no path for a one-line hotfix at 11pm.

None of this is malice. It's **friction** — the accumulated cost of a platform that's harder to use correctly than to use dangerously. Every unclear RBAC boundary, undocumented namespace convention, slow approval queue, and confusing error message is a small tax an operator pays to do their job. Eventually, enough people stop paying it, and the workaround becomes the normal way of working.

Standard security tooling doesn't see this coming. Vulnerability scanners look at images. Runtime tools watch syscalls and network calls. Observability platforms watch pods and nodes. All of them are built to catch what happens *after* the shortcut — the misconfigured role, the exposed workload, the incident. None of them are watching the person: where they hesitated, where they gave up, and where they quietly routed around the controls you built.

> Operator friction isn't a UX complaint. It's a leading indicator of exactly where your next security shortcut will come from.

## How Friction Intelligence solves it

Every action taken against your cluster already leaves a trace: the Kubernetes audit log. Most teams collect it for compliance and never look at it again. Healert reads it as behavior data instead — a record of how people actually work inside your platform, not just what happened to it.

A lightweight agent tails the audit log directly. It's self-hosted: no code runs inside your workloads, nothing calls out to the outside. Each event is classified against a configurable rules engine that looks for the patterns that precede a security shortcut — repeated permission denials right before an escalation, direct production edits that bypass GitOps, unusual `exec` sessions, RBAC grants far outside a role's normal baseline.

```
audit.k8s.io/v1  update  clusterrolebindings/emergency-access
user: operator@team-checkout · denied×3, then granted

  →  FRICTION SIGNAL: privilege escalation outside role baseline
```

Patterns like this get a friction score, surfaced per namespace, per team, per operator. The output isn't another threat alert competing for attention in a queue — it's a map. It shows a platform team exactly where the paved path breaks down: which role gets requested as an "emergency" grant every sprint, which team hits denied applies right before every release, where "just this once" access has quietly become the default.

Measured this way, friction becomes a leading indicator for both security exposure and platform usability — because they're the same problem, described from two different sides of the same audit log.

## The Backstage plugin

Healert is self-hosted by design: Docker Compose, your own Postgres, your own audit log source, nothing leaves your infrastructure. That's a deliberate choice for security-conscious teams, and especially for the regulated, and on-prem environments.

Backstage is where most platform teams already live — the service catalog, ownership data, docs, and golden paths all sit there. Putting friction visibility behind yet another dashboard would defeat the point, so Healert is being built as a Backstage plugin, surfacing friction scores next to the services and teams that already own them, inside the developer portal people already have open.

- See a friction score alongside each service in your existing catalog — no new tool to check.
- Drill into which teams are escalating privileges, bypassing GitOps, or hitting the same RBAC wall repeatedly.
- Run it entirely on infrastructure you control, alongside the rest of your Backstage instance.

The Backstage plugin is live now. Point Healert at your cluster's audit log and start seeing where friction is accumulating before it turns into an incident report.

---

We're calling this Friction Intelligence because it doesn't sit neatly inside "security" or "platform engineering" on their own — it lives in the seam between them, which is exactly where the shortcuts happen. 
