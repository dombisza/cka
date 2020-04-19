# cka
CKA exam preparation notes, in no particular order or logic. just some usefull stuff to practice.

**TMUX quickguide:** 
- https://linuxize.com/post/getting-started-with-tmux/

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

**create yaml templates fast**
	
```bash
kubectl create deployment nginx --image=nginx --dry-run -oyaml > deploy_nginx.yaml
kubectl expose deployment nginx --type=NodePort --name=nginx-service --dry-run -oyaml > nginx_service_for_deploy.yaml

kubectl run --generator=run-pod/v1 nginx --image=nginx -oyaml > pod_nginx.yaml
kubectl expose pod nginx --type=NodePort --name=nginx=service --dry-run -oyaml > nginx_service_for_pod.yaml

```

**static pods**

- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
- https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-2-b4f5c636eb4
		
```bash
ps auxfw | grep kubelet
--config=/var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
```

**daemonsets**

- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

**config maps**
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
**etcd backup**

- https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-3-2e7b44e89a3b

```bash
ETCDCTL_API=3 etcdctl help
ETCDCTL_API=3 etcdctl — endpoints=[ENDPOINT] — cacert=[CA CERT] — cert=[ETCD SERVER CERT] — key=[ETCD SERVER KEY] snapshot save [BACKUP FILE NAME]
kubectl describe pod etcd-master -n kube-system
```
**kubeadm upgrade**

- https://v1-16.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

```bash
#Get the version of the API server:

kubectl version --short

#View the version of kubelet:

kubectl describe nodes

#View the version of controller-manager pod:

kubectl get po [controller_pod_name] -o yaml -n kube-system

#Release the hold on versions of kubeadm and kubelet:

sudo apt-mark unhold kubeadm kubelet

#Install version 1.16.6 of kubeadm:

sudo apt install -y kubeadm=1.16.6-00

#Hold the version of kubeadm at 1.16.6:

sudo apt-mark hold kubeadm

#Verify the version of kubeadm:

kubeadm version

#Plan the upgrade of all the controller components:

sudo kubeadm upgrade plan

#Upgrade the controller components:

sudo kubeadm upgrade apply v1.16.6

#Release the hold on the version of kubectl:

sudo apt-mark unhold kubectl

#Upgrade kubectl:

sudo apt install -y kubectl=1.16.6-00

#Hold the version of kubectl at 1.16.6:

sudo apt-mark hold kubectl

#Upgrade the version of kubelet:

sudo apt install -y kubelet=1.16.6-00

#Hold the version of kubelet at 1.16.6:

sudo apt-mark hold kubelet

```


**rolling upgrade and rollback**

- https://kubernetes.io/docs/reference/kubectl/cheatsheet/ #Updating Resources
	
```bash
kubectl run nginx --image=nginx  --replicas=3
kubectl set image deploy/nginx nginx=nginx:1.9.1
kubectl rollout status deploy/nginx
kubectl rollout undo deploy/nginx
kubectl rollout status deploy/nginx
```
	
**taints and tolerations**

- https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

**affinity and anti-affinity**
- https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature

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
**nodeselector**

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

**fieldselectors and filtering**

- https://kubernetes.io/docs/reference/kubectl/cheatsheet/ 
- https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-4-17407899ef1a

```
kubectl get nodes -o jsonpath=’{.items[*].status.addresses[?(@.type==”ExternalIP”)].address}’
kubectl get services — sort-by=.metadata.name
kubectl get pods <pod-name> -o custom-columns=NAME:.metadata.name,RSRC:.metadata.resourceVersion
kubectl get pod -o jsonpath=’{.items[*].metadata.name}’
```

**kube-controll**

**cheatsheet**
#run though all of these at least 3 times
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/ 


**labelselectors**

**create pv**

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

**securityContext**
- https://linuxacademy.com/cp/courses/lesson/course/4019/lesson/6/module/327
	
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

**secrets**
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


**create service and ingress**

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

**network policies**

- https://kubernetes.io/docs/concepts/services-networking/network-policies/

**port forwarding**

- https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

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

**to check / read **

- https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
- https://kubernetes.io/docs/concepts/cluster-administration/cluster-administration-overview/#securing-a-cluster
- https://docs.google.com/spreadsheets/d/10NltoF_6y3mBwUzQ4bcQLQfCE1BWSgUDcJXy-Qp2JEU/edit#gid=0

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
**PRACTICE**

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

- Q: Label the worker node of your cluster with rack=qa.

- Q: Create a new namespace called "cloud9". Create a pod running k8s.gcr.io/liveness with a liveliness probe that uses httpGet to probe an endpoint path located at /cloud-health on port 8080.  The httpHeaders are name=Custom-Header and value=Awesome. The initial delay is 3 seconds and the period is 3.

- Q: Create a deployment with two replicas of nginx:1.7.9. The container listens on port 80. It should be named "web-dep" and be labeled with tier=frontend with an annotation of AppVersion=3.4. The containers must be running with the UID of 1000. Scale up the deployment to 10 replicas and perform a rolling update on it.

- Q: Configure a DaemonSet to run the image k8s.gcr.io/pause:2.0 in the cluster.

- Q: Configure the cluster to use 8.8.8.8 and 8.8.4.4 as upstream DNS servers. https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/

- Q: Figure out a way to create a pod with 3 replicas using the the nginx container that can have pods deployed 
on a worker node and the master node if needed. HINT taint

- Q: Copy all Kubernetes scheduler logs into a logs directory in your home directory.

- Q: Create a yaml file called db-secret.yaml for a secret called db-user-pass. The secret should have two fields: a username and password.  The username should be "superadmin" and the password should be "imamazing".

- Q: Create a ConfigMap called web-config that contains the following two entries: 'web_port' set to 'localhost:8080' 'external_url' set to 'reddit.com' Run a pod called web-config-pod running nginx, expose the configmap settings as environment variables inside the nginx container.

- Q: Create a namespace called awsdb in your cluster.  Create a pod called db-deploy that has one container running mysql image, and one container running nginx:1.7.9 In the same namespace create a pod called nginx-deploy with a single container running the image nginx:1.9.1.  Export the output of kubectl get pods for the awsdb namespace into a file called "pod-list.txt"

- Q: This requires having a cluster with 2 worker nodes Safely remove one node from the cluster.  Print the output of the node status into a file "worker-removed.txt". Reboot the worker node.   Print the output of node status showing worker unable to be scheduled to "rebooted-worker.txt" Now bring the node back into the cluster and schedule several nginx pods to it, print the get pods wide output showing at least  one pod is on the node you rebooted.

- Q: Create a deployment running nginx, mount a volume called "hostvolume" with a container volume mount at /tmp 
and mounted to the host at /data.  If the directory isn't there make sure it is created in the pod spec at run time.
Go into the container and create an empty file called "my-doc.txt" inside the /tmp directory.  On the worker node 
that it was scheduled to, go into the /data directory and output a list of the contents to list-output.txt showing 
the file exists.
