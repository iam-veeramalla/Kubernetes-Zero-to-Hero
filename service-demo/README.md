# Nginx ClusterIP Service

A `ClusterIP` Service provides a stable IP address and DNS name for access from inside the Kubernetes cluster. The `nginx-service` selector matches the `app: nginx` labels on the pods in the three-replica Nginx Deployment, and forwards TCP port 80 to each pod's port 80.

## Deploy

Run these commands from `service-demo/`:

```sh
kubectl apply -f deployment.yml
kubectl apply -f service.yml
```

## Verify

```sh
kubectl get deployments,pods,services
```

## Test locally

Forward the Service to localhost, then open `http://localhost:8080`:

```sh
kubectl port-forward service/nginx-service 8080:80
```

## Cleanup

Run from `service-demo/`:

```sh
kubectl delete -f service.yml
kubectl delete -f deployment.yml
```
