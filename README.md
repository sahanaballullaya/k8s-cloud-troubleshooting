# Kubernetes Troubleshooting Lab

Hands-on Kubernetes troubleshooting scenarios built in Minikube. Each scenario simulates a real pod failure, documents the diagnostic steps, and includes terminal screenshots as evidence.

Built as part of structured preparation for cloud support engineering roles.

---

## About

10 years of production support experience in managed service environments — Linux, AWS, incident management, RCA delivery, 97% CSAT over 5 years.

This repo documents my hands-on Kubernetes troubleshooting practice. Every scenario was run live in a local cluster. No simulated outputs — all screenshots are from real terminal sessions.

---

---

## Diagnostic methodology

Every scenario follows the same sequence regardless of the failure type:

```
1. Observe    kubectl get pods          — what is failing and how long
2. Identify   kubectl describe pod      — which layer, what exit code
3. Evidence   kubectl logs --previous   — what the app actually said
4. Node       kubectl describe node     — is the machine the problem
5. Fix cause  — not the symptom
6. Verify     kubectl get pods -w       — watch RESTARTS stay at 0
7. Document   — save logs and write RCA
```

The key instinct — never restart before you understand why it failed. Restarting without fixing the cause starts the loop again.

---

## Key learnings so far

**`--previous` flag** — without this flag you see empty logs from the freshly restarted container. `kubectl logs --previous` retrieves logs from the last crashed instance. That is where the evidence lives.

**Exit code before logs** — the exit code from `kubectl describe pod` tells you the category of failure before you read a single log line. 137 is a memory kill. 1 is an application crash. They point you in completely different directions.

**RESTARTS counter** — CrashLoopBackOff shows RESTARTS climbing. ImagePullBackOff stays at 0. That single number tells you whether the container ever started.

**Fix the cause not the symptom** — deleting and redeploying without fixing the underlying issue starts the loop again immediately.

---
