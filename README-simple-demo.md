# Simple Demo: Pod-to-Pod Communication via ClusterIP

Only 3 objects used — Deployment, Service, Pod. No ConfigMap, no volumes.

Files:
- `backend-deployment.yaml`  → 2 plain nginx pods (label: `app=backend`)
- `backend-service.yaml`     → ClusterIP Service named `backend-service`
- `frontend-deployment.yaml` → 1 pod with `curl` installed, used to call the backend

---

## 1. Deploy everything

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend-deployment.yaml
```

## 2. Show that each Pod has its own (unstable) IP

```bash
kubectl get pods -o wide
```

Point out the backend pods have two different IPs — and these IPs would
change if the pods were deleted and recreated.

## 3. Show the backend's stable ClusterIP

```bash
kubectl get svc backend-service
```

Note the `CLUSTER-IP` — this stays the same no matter which/how many
backend pods are running behind it.

## 4. Frontend Pod calls the Backend Pod — by Service name, not IP

```bash
kubectl get pods -l app=frontend        # copy the frontend pod name
kubectl exec -it <frontend-pod-name> -- curl http://backend-service
```

You'll see the nginx "Welcome to nginx!" HTML come back. The frontend pod
never used a Pod IP — it just used the name `backend-service`, and
Kubernetes' internal DNS + the ClusterIP handled the rest.

## 5. (Optional) Prove load balancing across the 2 backend pods

Run the curl a few times:

```bash
kubectl exec -it <frontend-pod-name> -- curl http://backend-service
kubectl exec -it <frontend-pod-name> -- curl http://backend-service
kubectl exec -it <frontend-pod-name> -- curl http://backend-service
kubectl exec -it <frontend-pod-name> -- curl http://backend-service
```

Then check both backend pods' logs:

```bash
kubectl get pods -l app=backend         # copy both backend pod names
kubectl logs <backend-pod-1>
kubectl logs <backend-pod-2>
```

You'll see the incoming requests split across both pods' access logs —
proof that the ClusterIP Service is load balancing between them.

---

## Key teaching point

> A Pod's IP is temporary. A Service's **ClusterIP is stable**, and any
> Pod in the cluster can reach it just by using the **Service name**
> (`backend-service`) — Kubernetes DNS + kube-proxy take care of routing
> and load balancing to the actual Pods behind it.

This is the same idea used in your NodePort/LoadBalancer demos — the only
difference is ClusterIP is reachable **only from inside the cluster**
(pod-to-pod), while NodePort/LoadBalancer also expose the Service
externally.
