
# **Goldilocks & Vertical Pod Autoscaler (VPA) Installation Guide**

This guide outlines the steps to install **Goldilocks** and **Vertical Pod Autoscaler (VPA)** on a Kubernetes cluster, which will help analyze resource utilization and provide recommendations for optimal resource requests and limits.

---

## **Prerequisites**

- A **Kubernetes** cluster running (local or cloud-based).
- **kubectl** configured to interact with your cluster.
- **Helm** installed on your system.

---

## **Step 1: Install Goldilocks**

Goldilocks helps Kubernetes users manage resource requests and limits by analyzing the resource consumption of pods and recommending optimal values. Follow these steps to install Goldilocks.

### 1.1 Add the Fairwinds Helm Chart Repository

Run the following command to add the Fairwinds chart repository:

```bash
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
```

### 1.2 Install Goldilocks Using Helm

You can install Goldilocks in the `goldilocks` namespace. If the namespace doesn’t exist, it will be created automatically.

```bash
kubectl create namespace goldilocks
helm install goldilocks fairwinds-stable/goldilocks --namespace goldilocks
```

This will install both the Goldilocks **controller** and the **dashboard** in your cluster.

### 1.3 Access Goldilocks Dashboard

Once the installation is complete, you can access the Goldilocks dashboard. You can expose the dashboard as a service, for example, using port forwarding:

```bash
kubectl port-forward svc/goldilocks-dashboard -n goldilocks 8080:80
```

Then navigate to the URL: `http://localhost:8080` in your browser to access the dashboard.

---

## **Step 2: Install Vertical Pod Autoscaler (VPA)**

The **Vertical Pod Autoscaler (VPA)** automatically adjusts resource requests and limits for running pods in a cluster. Goldilocks uses VPA to apply the resource recommendations it generates.

### 2.1 Install the VPA Custom Resource Definitions (CRDs)

The VPA requires some custom resource definitions (CRDs) before it can be installed. To install them, use the following command:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/vertical-pod-autoscaler/deploy/vpa-v1-crd.yaml
```

Note: Ensure the URL is correct and accessible. If you encounter a 404 or error, ensure you are using the correct version of the VPA CRDs.

### 2.2 Install the VPA Components

You can now install the VPA components using Helm.

First, add the Kubernetes autoscaler Helm chart repository:

```bash
helm repo add autoscaler https://kubernetes.github.io/autoscaler
```

Then, install the VPA with Helm:

```bash
helm install vpa autoscaler/vertical-pod-autoscaler --namespace kube-system
```

### 2.3 Verify VPA Installation

Ensure that VPA is installed correctly by checking the VPA pods:

```bash
kubectl get pods -n kube-system -l app=vertical-pod-autoscaler
```

---

## **Step 3: Label Namespaces for Goldilocks and VPA Integration**

Goldilocks works on namespaces that have a specific label. Ensure that your target namespaces are labeled to be included in Goldilocks' analysis and VPA recommendations.

### 3.1 Label the Namespace

To label a namespace for Goldilocks and VPA, run the following command:

```bash
kubectl label namespace <namespace-name> goldilocks.fairwinds.com/enabled=true
```

Replace `<namespace-name>` with the name of your target namespace.

---

## **Step 4: Viewing Resource Recommendations in the Goldilocks Dashboard**

Now that everything is installed, you can start viewing resource recommendations in the Goldilocks dashboard.

1. Go to `http://localhost:8080` (or the address you've configured for the dashboard).
2. You will see a list of your namespaces and their workloads.
3. Goldilocks will display recommendations for CPU and memory requests and limits for each workload in the labeled namespaces.

You can then use these recommendations to adjust resource requests and limits for your Kubernetes workloads.

---

## **Step 5: Additional Configuration and Troubleshooting**

### 5.1 If you face issues with "Error getting VPA data" in the dashboard:
- Make sure the **Vertical Pod Autoscaler (VPA)** is installed and running properly.
- Ensure that the namespace has the correct label (`goldilocks.fairwinds.com/enabled=true`).
- If the issue persists, try restarting the Goldilocks controller:

  ```bash
  kubectl rollout restart deployment goldilocks-controller -n goldilocks
  ```

### 5.2 Viewing VPA Recommendations Manually

If you want to view VPA recommendations directly:

```bash
kubectl get vpa <vpa-name> -n <namespace>
```

Replace `<vpa-name>` with the name of your VPA resource and `<namespace>` with the appropriate namespace.

---

## **Advantages of Using Goldilocks**

Goldilocks provides several **benefits** for optimizing resource management in your Kubernetes environment:

1. **Automated Resource Optimization**:
   - Goldilocks analyzes the actual resource usage of workloads and provides recommendations to optimize resource requests and limits. This helps ensure that your workloads are not over-provisioned or under-provisioned.

2. **Cost Efficiency**:
   - By optimizing the resource requests and limits, Goldilocks helps you avoid over-allocating resources, which can save costs in cloud environments by reducing unnecessary resource usage.

3. **Improved Cluster Performance**:
   - Properly set resource requests and limits help to avoid overloading nodes and ensure that workloads get the resources they need without starving other workloads of resources.

4. **Easy to Use**:
   - The **Goldilocks dashboard** provides a user-friendly interface that allows you to easily view resource usage and recommendations for your workloads.

5. **Integration with VPA**:
   - Goldilocks works alongside the **Vertical Pod Autoscaler (VPA)** to apply recommendations and automatically adjust resource requests and limits based on real-time data.

6. **Better Capacity Planning**:
   - Goldilocks gives insights into the current usage patterns of your workloads, helping you with better capacity planning for future deployments.

7. **Namespace-Level Analysis**:
   - Goldilocks allows you to perform resource analysis on a per-namespace level, making it easier to scale your workloads based on the specific resource needs of each namespace.

8. **Data-Driven Decision Making**:
   - The recommendations provided by Goldilocks are based on actual resource usage data, making them more reliable than estimates or manual calculations.

---

## **Conclusion**

With **Goldilocks** and **Vertical Pod Autoscaler (VPA)**, you now have an efficient solution for optimizing resource requests and limits in your Kubernetes cluster. Goldilocks helps you analyze existing resource usage and provide recommendations, while VPA can automatically adjust resources based on those recommendations.

You can continuously monitor workloads and make adjustments as needed, improving both **cost efficiency** and **resource management** within your Kubernetes environment.

---

### **Links:**
- [Goldilocks Documentation](https://goldilocks.docs.fairwinds.com/)
- [Vertical Pod Autoscaler Documentation](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)

---

