# Instructions for Deploying and Testing the ToDo Application

This document provides step-by-step instructions on how to apply the Kubernetes manifests and test the ToDo application.

## 1. Apply the Manifests

All Kubernetes manifests are located in the `.infrastructure` folder. You need to apply the namespace first, followed by the pods.

Run the following commands in your terminal:

```bash
# First, create the namespace
kubectl apply -f .infrastructure/namespace.yml

# Next, create the ToDo app pod and the busybox pod
kubectl apply -f .infrastructure/todoapp-pod.yml
kubectl apply -f .infrastructure/busybox.yml
```

Alternatively, you can apply all files in the folder at once:
```bash
kubectl apply -f .infrastructure/
```

Check if the pods are running correctly:
```bash
kubectl get pods -n todoapp
```

## 2. Test the Application using Port-Forward

You can access the ToDo application directly from your local machine using the `port-forward` command. Assuming your application runs on port 8080:

```bash
kubectl port-forward pod/todoapp-pod 8000:8000 -n todoapp
```

Now, open your web browser and navigate to `http://localhost:8000`. 

You can also test your health endpoints:
- `http://localhost:8000/readiness`
- `http://localhost:8000/health`

## 3. Test the Application using the Busybox Container

Because the ToDo app is deployed as a standalone Pod (without a Kubernetes Service), you will first need to find its internal IP address to communicate with it from the Busybox pod.

**Step A: Get the ToDo App Pod IP**
```bash
kubectl get pod todoapp-pod -n todoapp -o wide
```
Look for the `IP` column and copy the IP address (for example, `10.244.0.5`).

**Step B: Use `curl` from the Busybox Pod**
Run a command inside the running busybox container to send an HTTP request to the ToDo app pod. Replace `<POD_IP>` with the IP address you found in Step A, and `<busybox-pod-name>` with the actual name of your busybox pod:

```bash
kubectl exec -it <busybox-pod-name> -n todoapp -- curl http://<POD_IP>:8000
```

To test the specific probes you created:
```bash
kubectl exec -it <busybox-pod-name> -n todoapp -- curl http://<POD_IP>:8000/health
kubectl exec -it <busybox-pod-name> -n todoapp -- curl http://<POD_IP>:8000/readiness
```