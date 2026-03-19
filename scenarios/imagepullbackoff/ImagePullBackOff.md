# 🚨 Kubernetes Troubleshooting: ImagePullBackOff

---

## 📌 Overview

This scenario demonstrates how to troubleshoot and resolve an **ImagePullBackOff** error caused by an invalid container image in Kubernetes.

---

## 🎯 Objective

* Simulate a pod failure using an invalid image
* Identify the root cause using Kubernetes commands
* Fix the issue and validate recovery

---

## 🧩 Scenario Setup

The Deployment was created with an invalid image:

```yaml
image: nginx:not-a-real-tag
```

This causes Kubernetes to fail when pulling the image.

---

## 🛠️ Steps

### 1️⃣ Create Deployment with Invalid Image

📸 `screenshots/01-yaml-invalid-image.jpg`

* Created a Deployment with a non-existent image tag
* Applied using:

```bash
kubectl apply -f bad-image-demo.yaml
```

---

### 2️⃣ Observe Pod Failure

📸 `screenshots/02-pod-status-imagepullbackoff.jpg`

```bash
kubectl get pods
```

* Pod enters:

  * `ErrImagePull`
  * `ImagePullBackOff`

---

### 3️⃣ Diagnose the Issue

📸 `screenshots/03-describe-pod-image-error.jpg`

```bash
kubectl describe pod <pod-name>
```

* Found:

  * Failed to pull image
  * Image tag not found

---

### 4️⃣ Fix the YAML

📸 `screenshots/04-apply-fixed-yaml.jpg`

Updated:

```yaml
image: nginx:latest
```

---

### 5️⃣ Apply Fix & Rollout

📸 `screenshots/05-rollout-success.jpg`

```bash
kubectl apply -f bad-image-demo.yaml
kubectl rollout status deployment/bad-image-demo
```

* Deployment successfully rolled out

---

### 6️⃣ Verify Pod Running

📸 `screenshots/06-pod-running.jpg`

```bash
kubectl get pods
```

* Pod status → `Running`

---

## 🔍 Root Cause

Invalid container image:

```yaml
nginx:not-a-real-tag
```

Kubernetes could not pull the image from the registry.

---

## ✅ Resolution

* Corrected image tag
* Reapplied Deployment
* Verified successful rollout and pod health

---

## 🧠 Key Learnings

* `ImagePullBackOff` = image cannot be pulled
* `kubectl describe pod` is critical for debugging
* Kubernetes retries with backoff automatically
* Fix + apply triggers a rollout
* Pods are replaced automatically

---

## 📁 Project Structure

```
imagepullbackoff/
├── bad-image-demo.yaml
└── screenshots/
    ├── 01-yaml-invalid-image.jpg
    ├── 02-pod-status-imagepullbackoff.jpg
    ├── 03-describe-pod-image-error.jpg
    ├── 04-apply-fixed-yaml.jpg
    ├── 05-rollout-success.jpg
    └── 06-pod-running.jpg
```

---

## 🚀 Next Steps

* CrashLoopBackOff scenario
* Pending (resource constraints) scenario
* Add monitoring and alerting

---
