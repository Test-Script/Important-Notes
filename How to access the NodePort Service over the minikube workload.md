=========================================== Access the Service over the minikube IP ======================================

To access from the VM itself:
Step 1: minikube ip
Step 2: http://<that-ip>:32205
If not reachable:
minikube service deployment-loadbalancer-service --url

If you share the result of:
minikube ip
minikube service deployment-loadbalancer-service --url
I can tell you the exact working URL.

==========================================================================================================================

ACCESSING A KUBERNETES NodePort SERVICE
(Service: deployment-loadbalancer-service, Type: NodePort, Port: 80:32205)
--------------------------------------------------------------------------

1. SERVICE DETAILS

   * Service Type      : NodePort
   * Cluster-IP        : 10.108.226.138
   * NodePort          : 32205
   * Target Port       : 80
   * External-IP       : <none>
   * Namespace         : default (assumed)

2. HOW NodePort WORKS

   * Kubernetes exposes your application on **each worker node's IP**.
   * Access format:
     http://<Node-IP>:<NodePort>

3. FIND THE NODE IP
   Command:
   kubectl get nodes -o wide
   Example output:
   NAME         INTERNAL-IP
   minikube     192.168.49.2   (Minikube default)

4. ACCESS URL EXAMPLES
   If you are using **Minikube**, the node IP is almost always:
   192.168.49.2

   Therefore:
   [http://192.168.49.2:32205](http://192.168.49.2:32205)

   Or you can let minikube give you the correct IP:
   minikube ip
   Example:
   192.168.49.2
   Then use:
   http://$(minikube ip):32205

5. IMPORTANT NOTES

   * NodePort must be reachable from your host firewall.
   * If Pod is not running or readiness/liveness probes are failing, the endpoint will not respond.
   * If running on a VM (RHEL), ensure the firewall allows the port:
     firewall-cmd --add-port=32205/tcp --permanent
     firewall-cmd --reload

6. VERIFY SERVICE IS WORKING
   kubectl get svc deployment-loadbalancer-service
   kubectl describe svc deployment-loadbalancer-service
   kubectl get pods -o wide
   curl http://<node-ip>:32205

7. COMMON ISSUE IN MINIKUBE (PODMAN DRIVER)
   When using `--driver=podman`, `NodePort` works **only if Podman networking exposes the port**.
   If it does NOT work, run:
   minikube service deployment-loadbalancer-service --url
   This forwards the service directly to your host.

SUMMARY
Use this URL to access your service:
http://<Node-IP>:32205
For Minikube:
http://$(minikube ip):32205