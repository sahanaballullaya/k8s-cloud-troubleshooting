# ImagePullBackOff — Diagnostic Runbook

**Scope:** Kubernetes pod stuck in ImagePullBackOff  
**Environment:** Minikube lab  
**Author:** Sahana  
**Last updated:** 2026-03-18

---

## What is ImagePullBackOff

A pod enters ImagePullBackOff when Kubernetes cannot pull the container image from the registry. The pod never starts — no application runs, no logs are generated. Kubernetes retries with the same exponential backoff as CrashLoopBackOff: 10s, 20s, 40s, 80s, up to 5 minutes.

Two status values indicate this failure:

- `ErrImagePull` — the first failed pull attempt
- `ImagePullBackOff` — subsequent attempts after backoff kicks in

---

## Before you touch anything

1. Do not delete the pod until you have read the Events section
2. Save evidence before making any changes

---

## Diagnostic steps

### Step 1 — Confirm the failure

```bash
kubectl get pods
```

Expected output:

```
NAME               READY   STATUS             RESTARTS   AGE
bad-image-demo     0/1     ImagePullBackOff   0          4m
```

Key difference from CrashLoopBackOff — RESTARTS stays at 0. The container never started so there is nothing to restart.

---

### Step 2 — Read the Events

```bash
kubectl describe pod <pod-name>
```

Scroll to Events at the bottom. This is your only evidence — there are no logs because the container never ran.

```
Warning  Failed     kubelet  Failed to pull image "nginx:not-a-real-tag": 
                             Error response from daemon: manifest unknown
Warning  Failed     kubelet  Error: ErrImagePull
Warning  BackOff    kubelet  Back-off pulling image "nginx:not-a-real-tag"
```

The error message tells you exactly what went wrong — in this lab the image tag does not exist.

---

### Step 3 — Check what image was requested

```bash
kubectl get pod <pod-name> -o yaml | grep image
```

Output:

```
image: nginx:not-a-real-tag
```

This confirms the pod spec has the wrong image tag.

---

### Step 4 — Save your evidence

```bash
kubectl describe pod <pod-name> > pod-describe.txt
```

---

## Fix and verify

Update the image tag in the YAML file:

```yaml
image: nginx:latest
```

Apply the fix:

```bash
kubectl apply -f bad-image-demo.yaml
kubectl rollout status deployment/bad-image-demo
```

Confirm the pod is running:

```bash
kubectl get pods -w
```

Wait until STATUS shows Running and RESTARTS stays at 0.

---

## Key difference from CrashLoopBackOff

| | CrashLoopBackOff | ImagePullBackOff |
|---|---|---|
| Container started | Yes | No |
| Logs available | Yes — use --previous | No — container never ran |
| RESTARTS climbing | Yes | No — stays at 0 |
| Evidence location | Last State + logs | Events section only |

---

## Screenshots

### 01 — YAML with invalid image

![YAML invalid image](screenshots/01-yaml-invalid-image.jpg)

Deployment spec showing the incorrect image tag before the fix.

---

### 02 — Pod status ImagePullBackOff

![Pod status ImagePullBackOff](screenshots/02-pod-status-imagepullbackoff.jpg)

`kubectl get pods` showing ImagePullBackOff with RESTARTS at 0.

---

### 03 — Describe pod events

![Describe pod image error](screenshots/03-describe-pod-image-error.jpg)

`kubectl describe pod` showing the Failed to pull image event. This is the only evidence available — no logs exist.

---

### 04 — Fixed YAML applied

![Apply fixed YAML](screenshots/04-apply-fixed-yaml.jpg)

Updated deployment spec with correct image tag being applied.

---

### 05 — Rollout success

![Rollout success](screenshots/05-rollout-success.jpg)

`kubectl rollout status` confirming the deployment rolled out successfully.

---

### 06 — Pod running

![Pod running](screenshots/06-pod-running.jpg)

`kubectl get pods` showing STATUS: Running and RESTARTS: 0.

---

## Lab reproduction

```bash
# Start cluster
minikube start

# Deploy pod with invalid image
kubectl apply -f bad-image-demo.yaml

# Observe failure
kubectl get pods -w

# Diagnose
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml | grep image

# Save evidence
kubectl describe pod <pod-name> > pod-describe.txt

# Fix — update image tag in YAML then
kubectl apply -f bad-image-demo.yaml
kubectl rollout status deployment/bad-image-demo

# Verify
kubectl get pods -w
```

---

## Related scenarios

- [CrashLoopBackOff](../crashloopbackoff/crashloopbackoff-runbook.md) — container starts but crashes repeatedly
- [OOMKilled](../oomkilled/oomkilled-runbook.md) — exit code 137, memory limit exceeded
