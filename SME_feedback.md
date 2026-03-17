# SME Feedback: Container Deployment with Kubernetes
**Module:** Week 4, Class 8 — Container Deployment with Kubernetes  
**Reviewer:** Sumit Rahman  
**Documents Reviewed:** README.md (student-facing), Instructor Guide

---

## Overall Assessment

This is a solid module with accurate Kubernetes content and a well-chosen analogy
(K8s as the orchestra conductor). The architectural diagram is correct, the YAML
example is production-realistic, and the resource requests/limits section is the
most important technical concept in the module — it connects directly to the OOM
events that will appear in the lab data.

The primary structural issue is the lab section reference placeholder, which is now
resolved by the lab materials produced alongside this feedback. There are also several
opportunities to tighten the connection between the lesson's theoretical content and
the lab's hands-on tasks — particularly around readiness probes and the HPA.

---

## 1. Structural Issues

**Issue 1.1 — Section Numbering Jump**

The Table of Contents lists Sections 1–7, but Section 5 "Wrap-Up and Reflection" is
numbered 5 in the ToC and appears as such in the body — however, "Wrap-Up" is the
fifth body section while "Resources" is listed as Section 7 in the ToC. Counting the
sections in the body: there are six numbered sections (1–3 + Lab + Wrap-Up + Resources)
but the ToC shows seven entries with an "Introduction to AI Orchestration" as Section 1
that does not appear as a heading in the body. The body starts directly at "Benefits of
Containerization" (ToC Section 2). This is the same numbering drift issue as Lessons 5
and 7. The ToC and body should be reconciled.

**Issue 1.2 — Reflection Questions Are Numbered 1–3 But One Belongs in the Instructor Guide**

Reflection Question 3 ("How does Pytest serve as the first line of defense in an
automated CI/CD pipeline?") is a strong question but is a repeat of content from Lab 7
rather than material introduced in this module. It would be better placed as an
instructor debrief question connecting the two labs, rather than a student reflection
on new concepts.

---

## 2. Alignment with Syllabus and Competency Map

| Learning Objective | In README? | In Instructor Guide? | Notes |
|---|---|---|---|
| Explain benefits of containerisation for AI applications | ✅ Section 1 | ✅ Yes | Environment drift framing is accurate |
| Deploy and manage containers using Kubernetes clusters | ✅ Section 2 | ✅ Yes (live demo) | YAML example is production-realistic |
| Analyse deployment performance and scalability | ✅ Section 3 | ✅ Yes | Resource requests/limits is the key concept |

All three LOs are addressed. The weakest coverage is LO 3 — "analyse" is framed only
as defining throughput/latency rather than showing a worked analysis. The lab's
notebook component provides this analysis concretely.

---

## 3. Technical Content

**Issue 3.1 — Readiness Probe Not Mentioned in the Lesson**

The lesson covers Deployments, resource limits, and HPA (implicitly, via the scaling
discussion) but does not mention readiness or liveness probes. These are critical for
AI workloads specifically: a model pod that has started but is still loading weights
into GPU memory will accept connections, receive requests, and return 503 errors until
loading is complete. The readiness probe is the Kubernetes mechanism that prevents this.

The lab's `deployment.yaml` includes a readiness probe on `/health` — but students
arriving at the lab without any lecture coverage of probes will not understand why it
is there or what it does. Adding a one-paragraph callout to Section 2 would close this
gap.

**Issue 3.2 — HPA Not Formally Introduced**

Section 3 discusses "throughput" and "scalability" but does not introduce the
Horizontal Pod Autoscaler by name or explain its configuration. The instructor guide
references the HPA indirectly ("a model replica count of 5") and the lab's `hpa.yaml`
makes it concrete — but students without lecture coverage will not recognise what
the HPA YAML is doing.

The lesson's own reflection question ("Why might we prefer 5 small replicas over 1
huge pod?") is precisely the motivation for the HPA. Adding a brief HPA definition
and the `minReplicas`/`maxReplicas`/`averageUtilizationPercentage` fields to Section 3
would complete the theory-to-lab connection.

**Issue 3.3 — Resource Requests vs. Limits Distinction Needs Reinforcing**

Section 3 defines requests and limits correctly ("minimum guaranteed" vs. "hard
ceiling") but does not explain the failure mode: exceeding the memory limit causes
OOM eviction (the pod is killed), while exceeding the CPU limit causes CPU throttling
(the pod is slowed, not killed). This distinction matters for the lab's OOM analysis —
students who understand it can predict why Day 7 caused CrashLoopBackOff while high CPU
only caused latency increase.

**Issue 3.4 — `kubectl` Commands Absent from Student-Facing Content**

The lesson describes the Kubernetes workflow conceptually but does not show any
`kubectl` commands. Students who have never used Kubernetes will not know how to:
- Start Minikube: `minikube start --driver=docker`
- Apply manifests: `kubectl apply -f k8s/`
- Check pod status: `kubectl get pods`
- Read logs: `kubectl logs <pod-name>`

The instructor guide lists some of these implicitly (in the troubleshooting table), but
they should appear in the student-facing README so students can work through the lab
without needing to ask for help on basic `kubectl` operations.

---

## 4. Instructor Guide

**Issue 4.1 — Live Demo Scope is Too Broad for 40 Minutes**

The pacing guide allocates 40 minutes to "Kubernetes Objects & YAML" with a live demo
that covers: explaining Pods, Deployments, and Services; deploying a model to Minikube;
and showing K8s restarting a crashed container. In practice:
- `minikube start` alone takes 2–4 minutes
- Each `kubectl apply` + pod startup cycle takes 30–60 seconds
- Demonstrating a crash loop requires deliberately breaking a container

A more realistic 40-minute demo covers: reading and explaining the Deployment YAML
(10 min), applying it to a pre-started Minikube cluster (5 min), explaining
`kubectl get pods` and `kubectl describe pod` output (10 min), and walking through the
troubleshooting table for CrashLoopBackOff and ImagePullBackOff (15 min). The "crash
and restart" demonstration should be a screen recording rather than a live event.

**Issue 4.2 — Missing Debrief Connection to Lab 7**

The debrief questions are good but do not connect to the CI/CD pipeline from Lab 7.
The strongest connection is: the GitHub Actions workflow in Lab 8 adds a Kubernetes
deployment stage that only runs if the performance tests pass — making the readiness
probe and the performance gate part of the same reliability chain. A fourth debrief
question along the lines of: *"How does the readiness probe in Kubernetes complement
the performance tests in GitHub Actions? Are they redundant, or do they catch different
failure modes?"* would tie the two labs together effectively.

---

## 5. Theory-to-Lab Alignment Recommendations

**Recommendation 5.1 — Add Readiness Probe Callout to Section 2**  
A one-paragraph explanation of readiness vs. liveness probes, with the specific
`/health` endpoint pattern, would prevent the most common student confusion in the lab:
why is there a `/health` check in the YAML and why does the pod show "0/1 READY" for
the first 10 seconds?

**Recommendation 5.2 — Add HPA to Section 3**  
One paragraph + the `hpa.yaml` structure (minReplicas, maxReplicas, CPU target) would
scaffold the stretch task and make the Day 14 HPA ceiling event in the notebook data
interpretable without additional context.

**Recommendation 5.3 — Distinguish OOM Eviction from CPU Throttling in Section 3**  
The memory vs. CPU limit failure mode distinction (OOM kill vs. throttle) is the
conceptual key to the notebook's Part 1 analysis. Adding one sentence to the Resource
Requests and Limits paragraph would make students' notebook interpretations more precise.

---

## Summary of Action Items

| Priority | Issue | Type | Owner |
|---|---|---|---|
| High | Add readiness/liveness probe coverage to Section 2 | Fix | Content Author |
| High | Add HPA introduction to Section 3 | Fix | Content Author |
| Medium | Reconcile ToC numbering (Section 1 "Introduction" missing from body) | Fix | Content Author |
| Medium | Distinguish OOM eviction vs. CPU throttling in Resource Limits discussion | Recommendation | Content Author |
| Medium | Add `kubectl` command reference (start, apply, get, logs) to student README | Recommendation | Content Author |
| Medium | Narrow live demo scope to what fits in 40 minutes; pre-record crash loop demo | Recommendation | Instructor |
| Low | Move Reflection Question 3 (Pytest) to instructor debrief; replace with a Kubernetes-specific question | Fix | Content Author |
| Low | Add Lab 7 connection debrief question about readiness probe vs. performance test gate | Recommendation | Content Author |
