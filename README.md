# Container Deployment with Kubernetes

**Session Time:** 120 minutes

---

## Table of Contents

1. Benefits of Containerization for AI
2. Deploying and Managing Kubernetes Clusters
3. Analyzing Performance and Scalability
4. Lab: Testing Workflow Setup
5. Wrap-Up and Reflection
6. Resources

---

## Learning Objectives

By the end of this module, participants will be able to:

1. **Explain** the benefits of containerization for AI applications, focusing on reproducibility and dependency management.
2. **Deploy and manage** containers using Kubernetes clusters to ensure high availability of AI services.
3. **Analyze** deployment performance and scalability within a containerized environment to optimize resource allocation.

---

## Session Breakdown

| Segment | Topic                                   | Duration (minutes) |
|---------|-----------------------------------------|--------------------|
| 1       | The Case for Containers in AI           | 15                 |
| 2       | Kubernetes Architecture & Orchestration | 25                 |
| 3       | Scalability and Resource Optimization   | 20                 |
| 4       | Lab: Testing Workflow Setup             | 60                 |
**

---

## 1. Benefits of Containerization for AI

For professionals, the primary challenge in AI systems is "Environment Drift"—where a model performs differently in production than in development due to library versions (e.g., NumPy or PyTorch) or OS dependencies.

### Why Containerize AI?

* **Dependency Isolation:** Package specific CUDA versions or Python binaries without conflicting with the host system.
* **Portability:** Move a model from a local laptop to a massive GPU cloud provider without code changes.
* **Reproducibility:** Ensure that a model's inference results remain consistent across different lifecycle stages.

### Tool: Pytest (Validation)

Before a container is even built, **Pytest** is used to ensure the AI logic is sound.

**Practical Example:**
Using Pytest to check if an image classification model returns the correct shape before it is packaged into a Docker image.

```python
# test_model.py
import pytest
from model_loader import load_model

def test_inference_shape():
    model = load_model()
    dummy_input = ... 
    prediction = model.predict(dummy_input)
    assert prediction.shape == (1, 10), "Model output shape mismatch!"

```


---

## 2. Deploying and Managing Kubernetes Clusters

Kubernetes (K8s) acts as the "Operating System" for your data center, managing where and how your AI containers run. 
### Tool: Kubernetes & Minikube

* **Minikube:** A local Kubernetes tool that allows professionals to simulate a production cluster on a single machine.
* **Kubernetes:** The production-grade orchestrator that manages Pods (the smallest deployable units containing your containers).

### Essential `kubectl` Commands
To interact with your cluster and deploy models, you will use the following core commands:
* **Start Minikube:** `minikube start --driver=docker`
* **Apply manifests:** `kubectl apply -f k8s/`
* **Check pod status:** `kubectl get pods`
* **Read logs:** `kubectl logs <pod-name>`

### Ensuring Availability: Readiness and Liveness Probes
In AI workloads specifically, a model pod that has started but is still loading weights into GPU memory will accept connections, receive requests, and return 503 errors until loading is complete. 
* A **Readiness Probe** is the Kubernetes mechanism that prevents this. 
* By exposing a `/health` endpoint, Kubernetes knows not to route user traffic to the pod until it is fully initialized and ready to serve predictions.

### Architecture Diagram

```mermaid
graph TD
    subgraph Cluster [Kubernetes Cluster]
        API[API Server] --> Sch[Scheduler]
        Sch --> Node1[Worker Node 1]
        Sch --> Node2[Worker Node 2]
        Node1 --> Pod1[AI Model Pod A]
        Node1 --> Pod2[AI Model Pod B]
        Node2 --> Pod3[API Service Pod]
    end
    User((User/Engineer)) -->|kubectl| API

```
**

**Practical Example:**
Deploying an AI inference service using a `deployment.yaml` file to ensure 3 replicas of the model are always running.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-inference-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ai-model
  template:
    metadata:
      labels:
        app: ai-model
    spec:
      containers:
      - name: model-container
        image: my-registry/ai-model:v1.2
        ports:
        - containerPort: 8080

```

---

## 3. Analyzing Performance and Scalability

In AI Systems Engineering, performance is measured by throughput (requests per second) and latency, while scalability is the system's ability to handle increased load by adding resources.

### Automating Scalability: Horizontal Pod Autoscaler (HPA)
To handle variable inference loads, Kubernetes uses the Horizontal Pod Autoscaler (HPA).  Instead of guessing how many pods you need, you define a target metric.

For example, an `hpa.yaml` configuration defines the `minReplicas`, `maxReplicas`, and an `averageUtilizationPercentage` (like targeting 80% CPU usage). When inference traffic spikes and CPU usage exceeds 80%, HPA automatically spins up new pod replicas.

### Technical Detail: Resource Requests and Limits

To optimize performance, engineers must define **Resources**:

* **Requests:** The minimum CPU/RAM the AI container is guaranteed.
* **Limits:** The maximum CPU/RAM the container can consume. This prevents a "runaway" inference process from crashing the entire cluster. 

**Crucial Failure Modes:**
Exceeding these limits results in two distinct failure states:
* Exceeding the **memory limit** causes an OOM (Out of Memory) eviction, meaning the pod is killed.
* Exceeding the **CPU limit** causes CPU throttling, meaning the pod is slowed down, not killed.

### Tool: GitHub Actions (Continuous Deployment)

Automation ensures that performance checks are integrated into the deployment lifecycle. A GitHub Action triggers a deployment to a Kubernetes cluster only after the `Pytest` suite confirms the model meets latency requirements.

```mermaid
sequenceDiagram
    participant Dev as Engineer
    participant GHA as GitHub Actions
    participant K8s as Kubernetes Cluster
    
    Dev->>GHA: Push Code
    GHA->>GHA: Run Pytest (Performance Check)
    alt Tests Pass
        GHA->>K8s: Update Deployment Image
        K8s->>K8s: Rolling Update (New Pods)
        K8s-->>Dev: Deployment Success
    else Tests Fail
        GHA-->>Dev: Reject Deployment (Reliability Guard)
    end

```


---

## 4. Lab: Testing Workflow Setup

**Objective:** Integrate automated tests into a deployment pipeline using PyTest and GitHub Actions. This lab ensures learners can verify AI model and API performance consistently.

[Reference: Lab content]

---

## 5. Wrap-Up and Reflection

**Reflection Questions:**

1. Why is a rolling update in Kubernetes safer for a high-traffic AI model than a manual "delete and replace" approach?
2. How do resource limits in Kubernetes impact the cost-efficiency of an AI project?
3. If an AI model pod repeatedly enters a `CrashLoopBackOff` state, how might distinguishing between OOM eviction and CPU throttling guide your troubleshooting steps?

---

## 6. Resources

* **Kubernetes Basics:** [Official Kubernetes Documentation](https://kubernetes.io/docs/home/)
* **Local K8s Testing:** [Minikube Start Guide](https://minikube.sigs.k8s.io/docs/start/)
* **Testing for ML:** [Pytest Documentation](https://docs.pytest.org/en/stable/)
* **CI/CD Orchestration:** [GitHub Actions for Azure/AWS Kubernetes](https://github.com/features/actions)
* **Container Security:** [Best Practices for Dockerizing AI Models](https://www.google.com/search?q=https://www.docker.com/blog/how-to-dockerize-your-machine-learning-workflows/)
```
