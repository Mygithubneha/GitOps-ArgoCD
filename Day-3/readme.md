Dive into Argo CD: Hands-On with Installation and Usage 🚀
Hello, DevOps enthusiasts! Welcome back to our GitOps and Argo CD series. After exploring the fundamentals, we're now ready for a deeper dive into Argo CD. In this hands-on session, we'll cover everything from installation to deploying a sample application on a Kubernetes cluster. Let's get started! 🎉

Quick Recap 📝
Previously, we discussed:

GitOps Principles: Understanding the core concepts.
Argo CD Overview: How it fits into the GitOps ecosystem and why it's a popular choice.
Today, we’ll move from theory to practice. This guide is packed with step-by-step instructions and practical insights to get you up and running with Argo CD.

Getting Started with Argo CD 📚
Prerequisites
Ensure you have a Kubernetes cluster set up. For simplicity, we’ll use Minikube, a local Kubernetes setup.

Step 1: Set Up Your Kubernetes Cluster
First, verify your Minikube setup:


```json
minikube status
```
Ensure that your Minikube cluster is up and running. If not, start it with:

```javascript
minikube start
```

Step 2: Access Argo CD Documentation
The Argo CD documentation is your best friend. It’s detailed and provides everything you need to learn Argo CD comprehensively. For installation, navigate to the "Getting Started" section:

```json
https://argo-cd.readthedocs.io/en/stable/getting_started/
```

Step 3: Install Argo CD Using Plain Manifests
One of the easiest ways to install Argo CD is using plain manifests. Follow these steps:

Create a Namespace for Argo CD:


```json
kubectl create namespace argocd
```
GitOps-ArgoCD\Day-3\image-1.png

Install Argo CD Components:

```python
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
GitOps-ArgoCD\Day-3\image-2.png

This command will set up all necessary components, including custom resource definitions, services, service accounts, roles, role bindings, config maps, secrets, deployments, and network policies.

You can check the installation manifest to understand what’s being installed:


```python
curl https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Step 4: Verify the Installation
After applying the manifests, verify that all components are up and running:


```python
kubectl get pods -n argocd
```

You should see several pods running, including the API server, controller, and other components essential for Argo CD’s operation.

Using Argo CD 🖥️
Accessing the Argo CD UI
By default, Argo CD exposes a service that you can use to access its web UI. To access it:

Forward the Argo CD server port:


```python
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open your browser and navigate to:


```python
https://localhost:8080
```

Logging In
Use the default credentials to log in. The username is admin, and the password is the name of the argocd-initial-admin-secret secret:


```python
kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" -n argocd | base64 -d
```


Deploying a Sample Application
Now, let’s deploy a sample application using Argo CD. We will use a sample guestbook application from this GitHub repository:

Create an application manifest:

```python
yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/argoproj/argocd-example-apps'
    targetRevision: HEAD
    path: guestbook
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply the manifest:


```python
kubectl apply -f guestbook.yaml -n argocd

```
Managing Applications
You can manage your applications using both the Argo CD UI and CLI. The CLI is particularly useful for automation and scripting.

Using the CLI
Login to Argo CD CLI:

```python
argocd login localhost:8080
```

List applications:


```python
argocd app list
```
Get application details:


```python
argocd app get guestbook

```
Sync application:


```python
argocd app sync guestbook
```

Conclusion 🎉
Congratulations! You’ve now set up Argo CD, deployed a sample application, and learned to manage applications using both the UI and CLI. In future guides, we’ll explore more advanced topics, including integrating Helm charts and customizing configurations.

Stay tuned and happy DevOps-ing! 😊