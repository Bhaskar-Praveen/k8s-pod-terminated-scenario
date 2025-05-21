# Troubleshooting Investigation

## Issue Summary

After deploying the container image `nigelpoulton/web-app:1.0`, the pod enters a **Terminated** state with exit code `0`. This means the container process completed successfully, but exited instead of running continuously—a common scenario if the image runs a script or command that finishes immediately.

---

## Investigation Steps

1. **Check Pod Status**
    ```sh
    kubectl get pods
    kubectl describe pod <pod-name>
    ```
    - Use these commands to view the status, events, and details of the affected pod.

2. **View Pod Logs**
    ```sh
    kubectl logs <pod-name>
    ```
    - Inspect logs for error messages, completion notices, or unexpected exits.

3. **Analyze Deployment Manifest**
    - Confirm the container's entrypoint is meant to run indefinitely (e.g., a web server).
    - If the image simply runs a script and exits, the pod will terminate successfully with code `0`.

4. **Root Cause**
    - The container’s default command completes and exits.
    - This is normal for short-lived containers or demo images; persistent workloads (like web servers) should run as long-running processes.

---

## Resolution

- For web applications, ensure your Dockerfile launches a persistent process (e.g., `CMD ["python", "-m", "http.server"]` or similar).
- If the image is intended for demos or single-run jobs, this "Terminated (Completed)" status is expected.

---

## Key Takeaways

- **Understand the container's expected lifecycle:** Confirm whether the image is supposed to run persistently or exit.
- **First steps:** Use `kubectl logs` and `kubectl describe` for initial diagnosis.
- **Check the pod’s entrypoint/command:** Persistent apps need a long-running process.

---

## Useful Commands & Tips

- **List all pods in the cluster:**
    ```sh
    kubectl get pods
    ```
- **Get detailed information about a pod:**
    ```sh
    kubectl describe pod <pod-name>
    ```
- **Access the running container’s shell:**
    ```sh
    kubectl exec -it <pod-name> -- sh
    ```
    - Once inside, you can install tools (like `curl` using `apk add curl` in Alpine) to troubleshoot connectivity.
- **Check cluster and service IPs:**
    ```sh
    kubectl get svc
    ```
- **Check pods matching a specific label (e.g., sidecar):**
    ```sh
    kubectl get pods -l app=sidecar -o wide
    ```
- **Test HTTP connectivity:**
    ```sh
    curl http://<localhost IP>
    ```
    - If you get a response, your container is listening on port 80. If not accessible in the browser, further networking setup is needed.

### **Exposing Your Service**

**Option 1: Port Forwarding**
```sh
kubectl port-forward service/<service-name> 8080:80
```
- Access your app at [http://localhost:8080](http://localhost:8080)

**Option 2: Expose as LoadBalancer**
- Edit your service manifest to set `type: LoadBalancer`.
- Apply with:
    ```sh
    kubectl apply -f <service-manifest>.yml
    ```
- Check for external IP:
    ```sh
    kubectl get svc <service-name>
    ```
    - On Docker Desktop, EXTERNAL-IP may show as localhost; access via [http://localhost:80](http://localhost:80)

---

## Troubleshooting Checklist

- Ensure your pod is running and healthy:
    ```sh
    kubectl get pods -l app=<your-app-label>
    ```
- Verify your service selector matches your pod labels.
- Check service endpoints:
    ```sh
    kubectl describe svc <service-name>
    ```

---

**Pro Tip:**  
When troubleshooting, always confirm the expected behavior of both your container image and your Kubernetes manifests. Use the combination of `kubectl get`, `kubectl describe`, and `kubectl logs` to quickly zero in on issues.

---
