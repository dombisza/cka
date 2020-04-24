# cka
CKA exam preparation notes and practice questions.

- [Practice Questions and Tasks](https://github.com/dombisza/cka#practice)  
- [Application lifecycle management 8%](https://github.com/dombisza/cka/blob/master/README.md#application-lifecycle-management-8)  
- [Installation configuration validation 12% cluster maintenance 11%](https://github.com/dombisza/cka/blob/master/README.md#installation-configuration--validation-12--cluster-maintenance-11)  
- [Core concepts 19%](https://github.com/dombisza/cka/blob/master/README.md#core-concepts-19)  
- [Networking 11%](https://github.com/dombisza/cka/blob/master/README.md#networking-11)  
- [Scheduling 5%](https://github.com/dombisza/cka/blob/master/README.md#scheduling-5)  
- [Security 12%](https://github.com/dombisza/cka/blob/master/README.md#security-12)  
- [Logging monitoring 5%](https://github.com/dombisza/cka/blob/master/README.md#logging--monitoring-5)  
- [Troubleshooting 10%](https://github.com/dombisza/cka/blob/master/README.md#troubleshooting-10)  

The cluster I used for pacticing is a 3 node kubeadm bootstrapped cluster run on OTC ECS'es.

```bash
linux@sdombi-k8s-master:~$ kubectl get nodes -owide
NAME                 STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION   CONTAINER-RUNTIME
sdombi-k8s-master    Ready    master   10d   v1.15.7   192.168.1.96   <none>        Debian GNU/Linux 10 (buster)   4.19.0-8-amd64   docker://19.3.8
sdombi-k8s-worker1   Ready    <none>   10d   v1.15.7   192.168.1.7    <none>        Debian GNU/Linux 10 (buster)   4.19.0-8-amd64   docker://19.3.8
sdombi-k8s-worker2   Ready    <none>   10d   v1.15.7   192.168.1.27   <none>        Debian GNU/Linux 10 (buster)   4.19.0-8-amd64   docker://19.3.8

```
*add kubeadm cluster install notes here*

## Application Lifecycle Management 8%

https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#scaling-your-application  
https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/  

- bash autocompletion

```bash
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```

- create yaml templates fast
	
```bash
kubectl create deployment nginx --image=nginx --dry-run -oyaml > deploy_nginx.yaml
kubectl expose deployment nginx --type=NodePort --name=nginx-service --dry-run -oyaml > nginx_service_for_deploy.yaml

kubectl run --generator=run-pod/v1 nginx --image=nginx -oyaml > pod_nginx.yaml
kubectl expose pod nginx --type=NodePort --name=nginx=service --dry-run -oyaml > nginx_service_for_pod.yaml

```

- run busybox for testing and remove it automaticly on exit

```bash
kubectl run -it --rm --restart=Never busybox --image=busybox sh
```

- fieldselectors and filtering

https://kubernetes.io/docs/reference/kubectl/cheatsheet/  
https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-4-17407899ef1a

```
kubectl get nodes -o jsonpath=’{.items[*].status.addresses[?(@.type==”ExternalIP”)].address}’
kubectl get services — sort-by=.metadata.name
kubectl get pods <pod-name> -o custom-columns=NAME:.metadata.name,RSRC:.metadata.resourceVersion
kubectl get pod -o jsonpath=’{.items[*].metadata.name}’
```

- rolling upgrade and rollback

https://kubernetes.io/docs/reference/kubectl/cheatsheet/ #Updating Resources
	
```bash
kubectl run nginx --image=nginx  --replicas=3
kubectl set image deploy/nginx nginx=nginx:1.9.1
kubectl rollout status deploy/nginx
kubectl rollout undo deploy/nginx
kubectl rollout status deploy/nginx
```

## Installation, Configuration & Validation 12%** + **Cluster Maintenance 11%**

https://kubernetes.io/docs/concepts/cluster-administration/networking/  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/  
https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/  

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/  
https://kubernetes.io/docs/setup/  

- kubeadm upgrade

https://v1-16.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

```bash
##control plane
linux@sdombi-k8s-master:~$ sudo apt-cache policy kubeadm | grep 1.16.9-00
     1.16.9-00 500
linux@sdombi-k8s-master:~$ sudo apt-mark unhold kubeadm kubelet
Canceled hold on kubeadm.
Canceled hold on kubelet.
linux@sdombi-k8s-master:~$ sudo apt install -y kubeadm=1.16.9-00 >/dev/null

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

linux@sdombi-k8s-master:~$ sudo apt-mark hold kubeadm
linux@sdombi-k8s-master:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.9", GitCommit:"a17149e1a189050796ced469dbd78d380f2ed5ef", GitTreeState:"clean", BuildDate:"2020-04-16T11:42:30Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
linux@sdombi-k8s-master:~$ sudo kubeadm upgrade plan
...
COMPONENT   CURRENT       AVAILABLE
Kubelet     3 x v1.15.7   v1.16.9

Upgrade to the latest stable version:

COMPONENT            CURRENT    AVAILABLE
API Server           v1.15.11   v1.16.9
Controller Manager   v1.15.11   v1.16.9
Scheduler            v1.15.11   v1.16.9
Kube Proxy           v1.15.11   v1.16.9
CoreDNS              1.3.1      1.6.2
Etcd                 3.3.10     3.3.15-0

linux@sdombi-k8s-master:~$ sudo kubeadm upgrade apply v1.16.9
...
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.16.9". Enjoy!
linux@sdombi-k8s-master:~$ sudo apt install -y kubectl=1.16.9-00 >/dev/null

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

linux@sdombi-k8s-master:~$ sudo apt-mark hold kubectl
kubectl set on hold.
linux@sdombi-k8s-master:~$ sudo apt install kubelet=1.16.9-00
linux@sdombi-k8s-master:~$ kubelet --version
Kubernetes v1.16.9

##nodes
root@sdombi-k8s-worker1:/home/linux# apt-mark unhold kubeadm && apt install -y kubeadm=1.16.9-00 && apt-mark hold kubeadm
linux@sdombi-k8s-master:~$ kubectl drain sdombi-k8s-worker1 --ignore-daemonsets
linux@sdombi-k8s-worker1:~$ sudo kubeadm upgrade node
root@sdombi-k8s-worker1:/home/linux# apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.16.9-00 kubectl=1.16.9-00 && apt-mark hold kubelet kubectl
root@sdombi-k8s-worker1:/home/linux# systemctl restart kubelet
root@sdombi-k8s-worker1:/home/linux# kubelet --version
Kubernetes v1.16.9
linux@sdombi-k8s-master:~$ kubectl uncordon sdombi-k8s-worker1
node/sdombi-k8s-worker1 uncordoned

##do the same with node2

linux@sdombi-k8s-master:~$ kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
sdombi-k8s-master    Ready    master   12d   v1.16.9
sdombi-k8s-worker1   Ready    <none>   12d   v1.16.9
sdombi-k8s-worker2   Ready    <none>   12d   v1.16.9


```

- etcd backup

https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-3-2e7b44e89a3b

```bash
ETCDCTL_API=3 etcdctl help
ETCDCTL_API=3 etcdctl — endpoints=[ENDPOINT] — cacert=[CA CERT] — cert=[ETCD SERVER CERT] — key=[ETCD SERVER KEY] snapshot save [BACKUP FILE NAME]
kubectl describe pod etcd-master -n kube-system
```

## Core Concepts 19%**

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/  
https://kubernetes.io/docs/concepts/overview/components/  
https://kubernetes.io/docs/concepts/services-networking/service/  
https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/  

- config maps

```bash	
kubectl create configmap app-config --from-literal=key123=value123
```

```yaml
  containers:
  - image: nginx
    name: nginx
    env:
      - name: SPECIAL_APP_KEY
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: key123
```

## Networking 11%**
[Life of a Packet [I] - Michael Rubin, Google](https://www.youtube.com/watch?v=0Omvgd7Hg1I) #highly recommended!!!  
https://github.com/containernetworking/cni  
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/  
https://kubernetes.io/docs/concepts/services-networking/ingress/  
https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/  
https://kubernetes.io/docs/concepts/cluster-administration/networking/  

- network policies

https://kubernetes.io/docs/concepts/services-networking/network-policies/

- port forwarding

https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

```bash
kubectl create deployment nginx --image=nginx
kubectl get pods -l app=nginx

NAME                     READY   STATUS    RESTARTS   AGE
nginx-554b9c67f9-vt5rn   1/1     Running   0          10s

kubectl port-forward $POD_NAME 8080:80

Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

curl --head http://127.0.0.1:8080

kubectl logs $POD_NAME

```

- create service and ingress

```bash
kubectl run kubeserve2 --image=chadmcrowell/kubeserve2
kubectl expose deployment kubeserve2 --port 80 --target-port 8080 --type LoadBalancer
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: service-ingress
spec:
  rules:
  - host: kubeserve.example.com
    http:
      paths:
      - backend:
          serviceName: kubeserve2
          servicePort: 80
  - host: app.example.com
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
  - http:
      paths:
      - backend:
          serviceName: httpd
          servicePort: 80
```

## Scheduling 5%**

https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/  
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/  
https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/  

- static pods

https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/  
https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-2-b4f5c636eb4  
		
```bash
ps auxfw | grep kubelet
--config=/var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
```

- daemonsets

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

- taints and tolerations

https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

- affinity and anti-affinity

https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature

```bash
kubectl label nodes sdombi-k8s-worker1 flavor=2u8g
```

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: flavor
            operator: In
            values:
            - 2u8g
            - 4u8g
```
- nodeselector

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
      labels:
        env: test
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
      nodeSelector:
        disktype: ssd
```


## Security 12%**

https://kubernetes.io/docs/concepts/configuration/secret/  
https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/  
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/  
https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/  
https://kubernetes.io/docs/reference/access-authn-authz/rbac/  
https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/  

- NetworkPolicies
*deny all policy*
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
*allow port 9376 from app:web to app:hostnames*
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-netpolicy
spec:
  podSelector:
    matchLabels:
      app: hostnames
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - port: 9376
```

- ServiceAccounts, Secrets and Rolebinding

```bash
linux@sdombi-k8s-master:~$ kubectl create serviceaccount web
serviceaccount/web created
linux@sdombi-k8s-master:~$ kubectl get secrets web-token-6czm5
NAME              TYPE                                  DATA   AGE
web-token-6czm5   kubernetes.io/service-account-token   3      39s
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2020-04-21T13:56:07Z"
  name: web
  namespace: default
  resourceVersion: "1493503"
  selfLink: /api/v1/namespaces/default/serviceaccounts/web
  uid: ac79939b-4602-4a94-999b-dfa99e9c3cfa
secrets:
- name: web-token-6czm5
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: web-role-r
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

```
```bash
linux@sdombi-k8s-master:~$ kubectl apply -f web-roles.yaml
role.rbac.authorization.k8s.io/web-role-r created
linux@sdombi-k8s-master:~$ kubectl create rolebinding web-read --role=web-role-r --serviceaccount=default:web
rolebinding.rbac.authorization.k8s.io/web-read created

```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2020-04-21T14:05:41Z"
  name: web-read
  namespace: default
  resourceVersion: "1494315"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/default/rolebindings/web-read
  uid: 4c76fe63-6413-4c37-aa24-c31809e67b76
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: web-role-r
subjects:
- kind: ServiceAccount
  name: web
  namespace: default

```

- SecurityContext

https://linuxacademy.com/cp/courses/lesson/course/4019/lesson/6/module/327
	
```yaml
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-demo
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```


- secrets

```bash
kubectl create secret generic my-secret --from-literal=foo=bar -o yaml --dry-run > my-secret.yaml
kubectl create -f my-secret.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secrets-test-pod
spec:
  containers:
  - image: nginx
    name: test-container
    volumeMounts:
    - mountPath: /etc/secret/volume
      name: secret-volume
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
```

## Logging / Monitoring 5%**

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs  
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/#looking-at-logs  
https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods  
https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/  

## Storage 7%**

https://kubernetes.io/docs/concepts/storage/persistent-volumes/  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/  

- create pv

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

## Troubleshooting 10%**

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/  
https://kubernetes.io/docs/tasks/debug-application-cluster/determine-reason-pod-failure/  


**kube-controll**

**cheatsheet**
#run though all of these at least 3 times
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/ 


**labelselectors**

**to check / read **
- https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html
- https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
- https://kubernetes.io/docs/concepts/cluster-administration/cluster-administration-overview/#securing-a-cluster
- https://docs.google.com/spreadsheets/d/10NltoF_6y3mBwUzQ4bcQLQfCE1BWSgUDcJXy-Qp2JEU/edit#gid=0

**TMUX quickguide:** 
- https://linuxize.com/post/getting-started-with-tmux/  ## 'ctrl+b open-bracket

**openssl generate certificates for the cluster (.key, .crt, .csr, x509)**

```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt
openssl genrsa -out server.key 2048

###Create a config file for generating a Certificate Signing Request (CSR).
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = <country>
ST = <state>
L = <city>
O = <organization>
OU = <organization unit>
CN = <MASTER_IP>

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster
DNS.5 = kubernetes.default.svc.cluster.local
IP.1 = <MASTER_IP>
IP.2 = <MASTER_CLUSTER_IP>

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names

openssl req -new -key server.key -out server.csr -config csr.conf
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
-CAcreateserial -out server.crt -days 10000 \
-extensions v3_ext -extfile csr.conf
openssl x509  -noout -text -in ./server.crt
```

**some usefull kubectl commands**

```bash
$ kubectl cluster-info
$ kubectl get nodes
$ kubectl get componentstatuses
$ kubectl get pods -o wide --show-labels --all-namespaces
$ kubectl get svc  -o wide --show-labels --all-namespaces
kubectl explain
kubectl api-resources
kubectl get events
```
## PRACTICE

- Q: Create a yaml file called nginx-deploy.yaml for a deployment of three replicas of nginx, listening on the container's port 80. 
They should have the labels role=webserver and app=nginx. The deployment should be named nginx-deploy.
Expose the deployment with a load balancer and use a curl statement on the IP address of the load balancer 
to export the output to a file titled output.txt.

- Solution:

```bash
linux@sdombi-k8s-master:~$ kubectl create deployment nginx --image=nginx --dry-run -oyaml > nginx.yaml
```
edit the yaml, add[label, replicas, port]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
    role: webserver
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
status: {}
```

```bash
linux@sdombi-k8s-master:~$ kubectl apply -f nginx.yaml
linux@sdombi-k8s-master:~$ kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   2/3     3            2           9s

linux@sdombi-k8s-master:~$ kubectl expose deployment nginx-deploy --type=LoadBalancer --name=nginx-service
service/nginx-service exposed
linux@sdombi-k8s-master:~$ kubectl get svc -l=app=nginx
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   LoadBalancer   10.105.103.48   <pending>     80:32030/TCP   55s

```

- Q: Create a pod called "haz-docs" with an nginx image listening on port 80. 
Attach the pod to emptyDir storage, mounted to /tmp in the container. 
Connect to the pod and create a file with zero bytes in the /tmp directory called my-doc.txt. 

**solution**

```bash
linux@sdombi-k8s-master:~$ kubectl run haz-docs --generator=run-pod/v1 --image=nginx --port=80 --dry-run -oyaml > haz-docs.yaml
```

add volume
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: haz-docs
  name: haz-docs
spec:
  containers:
  - image: nginx
    name: haz-docs
    volumeMounts:
    - mountPath: /tmp
      name: tmpmount
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: tmpmount
    emptyDir: {}
status: {}

```

```bash
linux@sdombi-k8s-master:~$ kubectl apply -f haz-docs.yaml
pod/haz-docs created
linux@sdombi-k8s-master:~$ kubectl exec -it haz-docs touch /tmp/my-doc.txt
linux@sdombi-k8s-master:~$ kubectl exec -it haz-docs ls /tmp
my-doc.txt

```

- Q: Label the worker node of your cluster with rack=qa.

```bash
linux@sdombi-k8s-master:~$ kubectl label nodes sdombi-k8s-worker1 rack=qa
node/sdombi-k8s-worker1 labeled
```

- Q: Create a new namespace called "cloud9". Create a pod running k8s.gcr.io/liveness with a liveliness probe that uses httpGet to probe an endpoint path located at /cloud-health on port 8080.  The httpHeaders are name=Custom-Header and value=Awesome. The initial delay is 3 seconds and the period is 3.

**solution**

```bash
linux@sdombi-k8s-master:~$ kubectl create namespace cloud9
linux@sdombi-k8s-master:~$ kubectl run liveness --generator=run-pod/v1 --image=k8s.gcr.io/liveness -ncloud9 --dry-run -oyaml > liveness.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: liveness
  name: liveness
  namespace: cloud9
spec:
  containers:
  - image: k8s.gcr.io/liveness
    name: liveness
    resources: {}
    livenessProbe:
      httpGet:
        path: /cloud-health
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
linux@sdombi-k8s-master:~$ kubectl apply -f liveness.yaml
```


- Q: Create a deployment with two replicas of nginx:1.7.9. The container listens on port 80. It should be named "web-dep" and be labeled with tier=frontend with an annotation of AppVersion=3.4. The containers must be running with the UID of 1000. Scale up the deployment to 10 replicas and perform a rolling update on it.

**solution**

```bash
linux@sdombi-k8s-master:~$ kubectl create deployment nginx --image=nginx:1.7.9 --dry-run -oyaml > nginx179.yaml
```
add/set yaml[annotation,label,name,replicas,securitycontext,port]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
    tier: frontend
  name: web-dep
  annotations:
    AppVersion: 3.4
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      securityContext:
        runAsUser: 1000
      containers:
      - image: nginx:1.7.9
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}

```

```bash
linux@sdombi-k8s-master:~$ kubectl apply -f nginx179.yaml
linux@sdombi-k8s-master:~$ kubectl scale deployment --replicas=10 web-dep
deployment.extensions/web-dep scaled
linux@sdombi-k8s-master:~$ kubectl set image deploy/web-dep nginx=nginx:1.17
deployment.extensions/web-dep image updated
linux@sdombi-k8s-master:~$ kubectl rollout status deploy/web-dep
deployment "web-dep" successfully rolled out
linux@sdombi-k8s-master:~$ kubectl rollout undo deploy/web-dep
deployment.extensions/web-dep rolled back
linux@sdombi-k8s-master:~$ kubectl rollout status deploy/web-dep
Waiting for deployment "web-dep" rollout to finish: 9 of 10 updated replicas are available...
deployment "web-dep" successfully rolled out
linux@sdombi-k8s-master:~$ kubectl rollout status deploy/web-dep
deployment "web-dep" successfully rolled out

```

- Q: Configure a DaemonSet to run the image k8s.gcr.io/pause:2.0 in the cluster.

**solution**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: pause-daemon
  labels:
    app: paused
spec:
  selector:
    matchLabels:
      app: paused
  template:
    metadata:
      labels:
        app: paused
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: paused
        image: k8s.gcr.io/pause:2.0
      terminationGracePeriodSeconds: 30
```

```bash
linux@sdombi-k8s-master:~$ kubectl apply -f daemonset.yaml
daemonset.apps/pause-daemon created
linux@sdombi-k8s-master:~$ kubectl get daemonsets.
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
pause-daemon   3         3         3       3            3           <none>          4s
```

- Q: Configure the cluster to use 8.8.8.8 and 8.8.4.4 as upstream DNS servers. https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/

- Q: Figure out a way to create a pod with 3 replicas using the the nginx container that can have pods deployed 
on a worker node and the master node if needed. HINT taint

- Q: Copy all Kubernetes scheduler logs into a logs directory in your home directory.

**solution**

```bash
linux@sdombi-k8s-master:~$ kubectl logs `kubectl get pods -nkube-system | grep scheduler | awk '{print $1}'` -nkube-system > ~/scheduler.log
```

- Q: Create a yaml file called db-secret.yaml for a secret called db-user-pass. The secret should have two fields: a username and password.  The username should be "superadmin" and the password should be "imamazing".

**solution**

```bash
linux@sdombi-k8s-master:~$ echo -n 'superadmin' > username.txt
linux@sdombi-k8s-master:~$ echo -n 'pigbenispassword' > password.txt
linux@sdombi-k8s-master:~$ kubectl create secret generic db-secret --from-file=./username.txt --from-file=./password.txt --dry-run -oyaml > db-secret.yaml
linux@sdombi-k8s-master:~$ cat db-secret.yaml
apiVersion: v1
data:
  password.txt: aW1hbWF6aW5n
  username.txt: c3VwZXJhZG1pbg==
kind: Secret
metadata:
  creationTimestamp: null
  name: db-secret

```

- Q: Create a ConfigMap called web-config that contains the following two entries: 'web_port' set to 'localhost:8080' 'external_url' set to 'reddit.com' Run a pod called web-config-pod running nginx, expose the configmap settings as environment variables inside the nginx container.

- Q: Create a namespace called awsdb in your cluster.  Create a pod called db-deploy that has one container running mysql image, and one container running nginx:1.7.9 In the same namespace create a pod called nginx-deploy with a single container running the image nginx:1.9.1.  Export the output of kubectl get pods for the awsdb namespace into a file called "pod-list.txt"

- Q: This requires having a cluster with 2 worker nodes Safely remove one node from the cluster.  Print the output of the node status into a file "worker-removed.txt". Reboot the worker node.   Print the output of node status showing worker unable to be scheduled to "rebooted-worker.txt" Now bring the node back into the cluster and schedule several nginx pods to it, print the get pods wide output showing at least  one pod is on the node you rebooted.

```bash
linux@sdombi-k8s-master:~$ kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
NAME                            NODE
haz-docs                        sdombi-k8s-worker1
nginx-554b9c67f9-26qln          sdombi-k8s-worker2
nginx-554b9c67f9-6rd4n          sdombi-k8s-worker2
nginx-554b9c67f9-8dtwg          sdombi-k8s-worker1
nginx-554b9c67f9-bqxr8          sdombi-k8s-worker2
nginx-554b9c67f9-gflm4          sdombi-k8s-worker1
nginx-554b9c67f9-hp974          sdombi-k8s-worker1
nginx-554b9c67f9-hrm7t          sdombi-k8s-worker2
nginx-554b9c67f9-hzjrd          sdombi-k8s-worker2
nginx-554b9c67f9-jv86b          sdombi-k8s-worker1
nginx-554b9c67f9-p8mnm          sdombi-k8s-worker1
pause-daemon-6tjdf              sdombi-k8s-worker2
pause-daemon-9hp96              sdombi-k8s-worker1
pause-daemon-9sjn9              sdombi-k8s-master
static-web-sdombi-k8s-worker1   sdombi-k8s-worker1

linux@sdombi-k8s-master:~$ kubectl drain sdombi-k8s-worker1 --ignore-daemonsets

linux@sdombi-k8s-master:~$ kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
NAME                            NODE
nginx-554b9c67f9-26qln          sdombi-k8s-worker2
nginx-554b9c67f9-2mw6s          sdombi-k8s-worker2
nginx-554b9c67f9-6rd4n          sdombi-k8s-worker2
nginx-554b9c67f9-9cfht          sdombi-k8s-worker2
nginx-554b9c67f9-bbxl6          sdombi-k8s-worker2
nginx-554b9c67f9-bqxr8          sdombi-k8s-worker2
nginx-554b9c67f9-bxfqv          sdombi-k8s-worker2
nginx-554b9c67f9-hrm7t          sdombi-k8s-worker2
nginx-554b9c67f9-hzjrd          sdombi-k8s-worker2
nginx-554b9c67f9-lrr49          sdombi-k8s-worker2
pause-daemon-6tjdf              sdombi-k8s-worker2
pause-daemon-9hp96              sdombi-k8s-worker1
pause-daemon-9sjn9              sdombi-k8s-master
static-web-sdombi-k8s-worker1   sdombi-k8s-worker1

linux@sdombi-k8s-master:~$ kubectl get nodes
NAME                 STATUS                     ROLES    AGE   VERSION
sdombi-k8s-master    Ready                      master   11d   v1.15.7
sdombi-k8s-worker1   Ready,SchedulingDisabled   <none>   11d   v1.15.7
sdombi-k8s-worker2   Ready                      <none>   11d   v1.15.7

#REBOOT node01

linux@sdombi-k8s-master:~$ kubectl uncordon sdombi-k8s-worker1
node/sdombi-k8s-worker1 uncordoned
linux@sdombi-k8s-master:~$ kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
NAME                            NODE
nginx-554b9c67f9-26qln          sdombi-k8s-worker2
nginx-554b9c67f9-2mw6s          sdombi-k8s-worker2
nginx-554b9c67f9-6rd4n          sdombi-k8s-worker2
nginx-554b9c67f9-9cfht          sdombi-k8s-worker2
nginx-554b9c67f9-bbxl6          sdombi-k8s-worker2
nginx-554b9c67f9-bqxr8          sdombi-k8s-worker2
nginx-554b9c67f9-bxfqv          sdombi-k8s-worker2
nginx-554b9c67f9-hrm7t          sdombi-k8s-worker2
nginx-554b9c67f9-hzjrd          sdombi-k8s-worker2
nginx-554b9c67f9-lrr49          sdombi-k8s-worker2
pause-daemon-6tjdf              sdombi-k8s-worker2
pause-daemon-9hp96              sdombi-k8s-worker1
pause-daemon-9sjn9              sdombi-k8s-master
static-web-sdombi-k8s-worker1   sdombi-k8s-worker1

# my only idea was to taint worker2 to stop scheduling and reschedule 5 pods to worker1 to keep it balanced

linux@sdombi-k8s-master:~$ kubectl taint nodes sdombi-k8s-worker2 stop=true:NoSchedule
node/sdombi-k8s-worker2 tainted

linux@sdombi-k8s-master:~$ kubectl get pods -o custom-columns=NAME:.metadata.name | grep nginx | tail -5 | while read x; do kubectl delete pod $x; done
pod "nginx-554b9c67f9-bqxr8" deleted
pod "nginx-554b9c67f9-bxfqv" deleted
pod "nginx-554b9c67f9-hrm7t" deleted
pod "nginx-554b9c67f9-hzjrd" deleted
pod "nginx-554b9c67f9-lrr49" deleted

linux@sdombi-k8s-master:~$ kubectl taint node sdombi-k8s-worker2 stop:NoSchedule-
node/sdombi-k8s-worker2 untainted
linux@sdombi-k8s-master:~$ kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName            NAME                            NODE
nginx-554b9c67f9-26qln          sdombi-k8s-worker2
nginx-554b9c67f9-2mw6s          sdombi-k8s-worker2
nginx-554b9c67f9-594q8          sdombi-k8s-worker1
nginx-554b9c67f9-5gw5m          sdombi-k8s-worker1
nginx-554b9c67f9-6rd4n          sdombi-k8s-worker2
nginx-554b9c67f9-8qw6z          sdombi-k8s-worker1
nginx-554b9c67f9-9cfht          sdombi-k8s-worker2
nginx-554b9c67f9-bbxl6          sdombi-k8s-worker2
nginx-554b9c67f9-l4fnd          sdombi-k8s-worker1
nginx-554b9c67f9-mjpq6          sdombi-k8s-worker1
pause-daemon-6tjdf              sdombi-k8s-worker2
pause-daemon-9hp96              sdombi-k8s-worker1
pause-daemon-9sjn9              sdombi-k8s-master
static-web-sdombi-k8s-worker1   sdombi-k8s-worker1

```

- Q: Create a deployment running nginx, mount a volume called "hostvolume" with a container volume mount at /tmp 
and mounted to the host at /data.  If the directory isn't there make sure it is created in the pod spec at run time.
Go into the container and create an empty file called "my-doc.txt" inside the /tmp directory.  On the worker node 
that it was scheduled to, go into the /data directory and output a list of the contents to list-output.txt showing 
the file exists.
