## Deploy the Kubernetes objects. 
In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

- Create a deployment for the [nginx](https://nginx.org/en/) web server:

```
$ kubectl run nginx --image=nginx:alpine
```

- List the pod created by the `nginx` deployment:

```
$ kubectl get pods -l run=nginx
```
> output
```
NAME                     READY     STATUS    RESTARTS   AGE
nginx-65899c769f-xkfcn   1/1       Running   0          15s
```

## Port Forwarding [On Master node]

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

- Retrieve the full name of the `nginx` pod:

```
$ POD_NAME=$(kubectl get pods -l run=nginx -o jsonpath="{.items[0].metadata.name}")
```

- Forward port `8088` on your local machine to port `80` of the `nginx` pod:

```
$ kubectl port-forward $POD_NAME 8088:80
```


- In a new terminal make an HTTP request using the forwarding address:

```
$ curl --head 127.0.0.1:8088
```
> output
```
Handling connection for 8088
HTTP/1.1 200 OK
Server: nginx/1.15.2
Date: Wed, 29 Aug 2018 03:59:29 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 24 Jul 2018 13:02:29 GMT
Connection: keep-alive
ETag: "5b572365-264"
Accept-Ranges: bytes
```

- Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```
Forwarding from 127.0.0.1:8088 -> 80
Forwarding from [::1]:8088 -> 80
Handling connection for 8088
^C
```

## Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

- Print the `nginx` pod logs:

```
$ kubectl logs $POD_NAME
```
> output
```
127.0.0.1 - - [29/Aug/2018:03:59:29 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.47.0" "-"
```

### Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

- Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```
$ kubectl exec -ti $POD_NAME -- nginx -v
```
> output
```
nginx version: nginx/1.15.2
```

## Services

In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the `nginx` deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service:

```
$ kubectl expose deployment nginx --port 80 --type NodePort
```

- Retrieve the node port assigned to the `nginx` service:

```
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
  
$ echo $NODE_PORT  
```      

- Get the node on which our nginc pod is running.
```
$ kubectl get pod $POD_NAME -o wide
```
> output
```
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-65899c769f-bfb6p   1/1       Running   0          20m       10.244.0.3   worker-2
```
You can access the nginx at `$NODE_PORT` port of worker2's Public IP.


## DNS [Not working]

## Verification

- Create a `busybox` deployment:

```
$ kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```

- List the pod created by the `busybox` deployment:

```
$ kubectl get pods -l run=busybox
```
> output
```
NAME                      READY   STATUS    RESTARTS   AGE
busybox-bd8fb7cbd-vflm9   1/1     Running   0          10s
```

- Retrieve the full name of the `busybox` pod:

```
$ POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

- Get the list of the Services

```
$ kubectl get svc
```
> output
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP        1h
nginx        NodePort    10.32.0.40   <none>        80:30625/TCP   43m
```

- Execute a DNS lookup for the `kubernetes` service inside the `busybox` pod:

```
$ kubectl exec -ti $POD_NAME -- nslookup kubernetes
```
> output
```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local

```

- Execute a DNS lookup for the `nginx` service inside the `busybox` pod:

```
$ kubectl exec -ti $POD_NAME -- nslookup nginx
```
> output
```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx
Address 1: 10.32.0.40 nginx.default.svc.cluster.local

```