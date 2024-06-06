Deploying Applications to Multiple Kubernetes Clusters with Argo CD 🌟

Hello, everyone! Today, we'll explore how to deploy applications across multiple Kubernetes clusters simultaneously using Argo CD. Argo CD, a GitOps-based tool, has become immensely popular in the continuous delivery space. 🌍

Traditional CI/CD Pipelines vs. GitOps
In legacy CI/CD pipelines, continuous integration (CI) and continuous delivery (CD) are often managed with shell scripts, Python scripts, Ansible, or CI orchestrator plugins. While easy to implement, these methods have significant drawbacks. That's where GitOps, and specifically Argo CD, come in! 🚀

Why Multi-Cluster Deployment?

Organizations typically use multiple Kubernetes clusters for various environments (development, QA, production, etc.). When a new version of an application needs deployment, it’s essential to update all relevant clusters efficiently and securely. Argo CD simplifies this process by ensuring that all changes are version-controlled and automatically deployed across clusters. 🌐

Setting Up Multi-Cluster Deployment

Before we dive in, ensure you have multiple Kubernetes clusters. Here's a quick overview of our setup:

Hub Cluster: Centralized control where Argo CD is installed.

Spoke Clusters: Additional clusters managed by the Hub.

To create these clusters using eksctl:


```javascript
eksctl create cluster --name=hub-cluster
eksctl create cluster --name=spoke-cluster-1
eksctl create cluster --name=spoke-cluster-2
```

Argo CD Deployment Model

Argo CD can be deployed in two main models:

Standalone Model: Each Kubernetes cluster has its own Argo CD instance.

Hub-Spoke Model: One central Argo CD instance in the Hub Cluster manages deployments across multiple clusters.

For larger organizations with centralized DevOps teams, the Hub-Spoke model is often more efficient. It simplifies management, upgrades, and disaster recovery. However, it requires high availability and robust resource configuration. 💡

Deploying with Argo CD

With Argo CD, your Git repository becomes the single source of truth. All Kubernetes manifests (ConfigMaps, Secrets, Deployments, etc.) are stored in Git. Argo CD monitors both the Git repository and the Kubernetes cluster, ensuring they remain in sync. If any manual changes occur in the cluster, Argo CD will revert them to match the repository state, maintaining consistency and security. 🔒

Example Configuration

Here’s a simplified setup for deploying an application (guest-book) across multiple clusters:

Create Kubernetes Clusters:


```javascript
eksctl create cluster --name=hub-cluster
eksctl create cluster --name=spoke-cluster-1
eksctl create cluster --name=spoke-cluster-2
```

Deploy Argo CD in the Hub Cluster:


```javascript
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


Configure Argo CD for Multi-Cluster Deployment:

yaml
Copy code
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  repositories: |
    - url: https://github.com/your-repo/argocd-example-apps
  clusterCredentials: |
    - name: spoke-cluster-1
      server: https://<spoke-cluster-1-api-server>
      config: |
        bearerToken: <token>
    - name: spoke-cluster-2
      server: https://<spoke-cluster-2-api-server>
      config: |
        bearerToken: <token>

Deploy the Application:

```javascript

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guest-book
  namespace: argocd
spec:
  destination:
    namespace: default
    server: https://<spoke-cluster-1-api-server>
  source:
    repoURL: https://github.com/your-repo/argocd-example-apps
    targetRevision: HEAD
    path: guest-book
```
Benefits of Argo CD

Automatic Reconciliation: Keeps your clusters in sync with Git.

Enhanced Security: Reverts unauthorized changes.
Scalability: Manages multiple clusters from a central point.
By adopting Argo CD and the GitOps approach, you ensure your deployments are secure, consistent, and scalable. Happy deploying! 😊

For more detailed steps, visit the Argo CD example project. 🌟

In this extension, we'll cover troubleshooting issues, installing Argo CD, adding clusters, and deploying applications across multiple clusters.

Troubleshooting CloudFormation Issues
During the creation of our EKS clusters, you might encounter issues with CloudFormation stacks. For example, if a stack already exists, you need to delete it before proceeding. Here's how to handle such scenarios:

Identify the Issue:
If you encounter an error during cluster creation, check the CloudFormation stacks in the AWS console. Navigate to the region where your clusters are being created (e.g., US-West-1).

Delete the Stack:
If a stack exists that is causing issues, delete it. This can usually be done from the CloudFormation console.

Retry the Cluster Creation:
After the problematic stack is deleted, retry the creation process for the affected cluster.

Installing Argo CD on the Hub Cluster
Once your Hub and Spoke clusters are ready, the next step is to install Argo CD on the Hub cluster.

Install Argo CD:


```javascript
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify Installation:

Check that all Argo CD components are running:
```javascript

kubectl get pods -n argocd
```
Access Argo CD UI:

Change the service type to NodePort to access the Argo CD UI without using Ingress:


```javascript
kubectl edit svc argocd-server -n argocd
```

Change type: ClusterIP to type: NodePort.

Open Security Groups:
Ensure that the security groups for your EC2 instances allow access to the NodePort range.

Login to Argo CD:
Retrieve the initial admin password and login:


```javascript
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

Use this password to log in to the Argo CD UI.

Adding Spoke Clusters to Argo CD
To manage deployments across multiple clusters, add each Spoke cluster to Argo CD:

Login to Argo CD CLI:


```javascript
argocd login <ARGO_CD_SERVER>
```

Add Clusters:


```javascript
argocd cluster add <SPOKE_CLUSTER_CONTEXT>
```

Repeat this for each Spoke cluster.

Verify Clusters:
In the Argo CD UI, navigate to Settings > Clusters to see the added clusters.

Deploying Applications

Now that Argo CD is set up and the clusters are added, deploy applications using a Git repository.

Create an Application in Argo CD:

Click on New App.
Fill in the necessary details such as the application name, project, sync policy, and repository details.
Select the destination cluster and namespace.
Monitor Deployment:
The application should be synced and deployed to the specified clusters. Check the Argo CD UI for the status.

Update Application:
Any changes pushed to the Git repository will automatically trigger deployments across the clusters.

Conclusion
By following these steps, you can successfully set up a multi-cluster EKS environment with a centralized Argo CD management system. This setup allows for efficient and scalable deployments across multiple Kubernetes clusters, leveraging the power of GitOps for continuous delivery.





