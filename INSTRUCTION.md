# INSTRUCTION

## 1. Deploy the application

Apply the namespace and Kubernetes manifests:

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/nodeport.yml
```

Check the deployment and autoscaler:

```bash
kubectl get deployments -n mateapp
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
kubectl get services -n mateapp
```

## 2. Resource requests and limits

The Deployment requests `100m` CPU and `128Mi` memory per container. These values provide a modest baseline for a small Django application while allowing Kubernetes to schedule the pods based on their resource requirements.

The limits are `500m` CPU and `256Mi` memory per container. They cap resource consumption and help prevent the application from using excessive node resources. These are initial estimates and can be adjusted after observing actual workload metrics.

## 3. Deployment strategy

The Deployment uses `RollingUpdate` with `maxSurge: 1` and `maxUnavailable: 0`.

`maxSurge: 1` allows Kubernetes to create one additional pod temporarily during an update. `maxUnavailable: 0` prevents Kubernetes from intentionally reducing the number of available replicas during the rollout. This prioritizes application availability while new pods start and pass readiness checks.

The Deployment starts with two replicas to provide basic redundancy.

## 4. Horizontal Pod Autoscaler

The HPA targets the `todoapp` Deployment in the `mateapp` namespace. It maintains a minimum of two replicas and can scale up to five replicas.

Scaling targets are set to average CPU utilization of 70% and average memory utilization of 80%, calculated relative to the resource requests. These targets allow the application to scale when resource demand rises while avoiding unnecessary scaling at low load.

Metrics Server must be installed and working in the cluster for CPU and memory metrics to be available. Without metrics, the HPA cannot make scaling decisions based on resource utilization.

## 5. Access the application

The application is exposed using the NodePort Service defined in `.infrastructure/nodeport.yml`.

Check the Service and the assigned NodePort:

```bash
kubectl get service -n mateapp
kubectl get nodes -o wide
```

Open the application using:

```text
http://<NODE-IP>:<NODE-PORT>/
```

Replace `<NODE-IP>` with the Kubernetes node IP and `<NODE-PORT>` with the NodePort shown by the Service.
