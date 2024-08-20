# Kyverno

## Introduction

Kyverno is a policy engine designed for Kubernetes. It allows cluster administrators to validate, mutate, and generate resources in a Kubernetes cluster. Kyverno policies are Kubernetes resources that are managed and applied in the same way as other resources in a Kubernetes cluster.

More information about Kyverno can be found at [Kyverno.io](https://kyverno.io/).

This README will guide you through the following steps:

- Create a Kubernetes cluster with k3d
- Add Kyverno and 5 policies (2 admission, 2 mutation, 1 generate)
- Add a web application as support (Whoami)
- Test the policies

## Installation

### Setup a Kubernetes cluster with k3d

```bash
k3d cluster create -c k3d.yaml
```

This will create a Kubernetes cluster using the configuration in the `k3d.yaml` file. The cluster will have 1 control-plane node, 2 worker nodes and a load balancer.

Once the cluster is created, let's create the namespaces:

```bash
kubectl apply -f namespaces
```

### Install Kyverno

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm install kyverno --namespace kyverno kyverno/kyverno --wait
```

### Apply Kyverno policies

```bash
kubectl apply -f kyverno/policies/
```

### Deploy the web application

```bash
kubectl apply -f app/
```

To see the application, you can use a port-forward to the service. The application will be available at `http://localhost:8080`. It will return the hostname of the pod.

> Warning: The port forward will always return the same hostname because he is always pointing to the first available pod.

```bash
kubectl port-forward svc/whoami-service 8080:80 -n app
```

## Check Kyverno policies

### Admission policies

- **block-privileged-containers.yaml**: This policy blocks privileged containers in the `app` namespace.

We can see in the deployment that the `privileged` flag is set to `true`:

```bash
kubectl get deploy whoami-deployment -n app -o jsonpath='{.spec.template.spec.containers[0].securityContext.privileged}'
```

> To test the policy, you can delete/set the deployment to `privileged: true` and see the result.

- **restrict-pod-resource-requests-limits.yaml**: This policy restricts the resource requests and limits for the `app` namespace.

We can see in the deployment that the `requests` and `limits` are set to `1`:

```bash
kubectl get deploy whoami-deployment -n app -o jsonpath='{.spec.template.spec.containers[0].resources.requests.cpu}'
kubectl get deploy whoami-deployment -n app -o jsonpath='{.spec.template.spec.containers[0].resources.limits.cpu}'
```

> To test the policy, you can delete the deployment limits and requests and see the result.

### Mutation policies

- **add-default-labels.yaml**: This policy adds default labels to the resources in the `app` namespace.

We can see in the deployment that the labels are set to `environment: production`:

```bash
kubectl get deploy whoami-deployment -n app -o jsonpath='{.spec.template.metadata.labels}'
```

> As you can see, the labels are not set in the deployment, but the policy will add them.

- **add-security-context.yaml**: This policy adds a security context to the resources in the `app` namespace.

We can see in the deployment that the security context is set to `runAsNonRoot: true` and `runAsUser: 1000`:

```bash
kubectl get deploy whoami-deployment -n app -o jsonpath='{.spec.template.spec.securityContext.runAsNonRoot}'
kubectl get deploy whoami-deployment -n app -o jsonpath='{.spec.template.spec.securityContext.runAsUser}'
```

> As you can see, the labels are not set in the deployment, but the policy will add them.

### Generate policies

- **generate-default-network-policy.yaml**: This policy generates a default network policy for the `app` namespace.

We can see the network policy generated:

```bash
kubectl get networkpolicy default-deny -n app -o yaml
```

> As you can see, the network policy is not present in the `app` folder, but the policy will generate it.
