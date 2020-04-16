# cka
CKA exam preparation notes by szabolcs dombi, in no particular order or logic. just some usefull stuff to practice.

**TMUX quickguide:** https://linuxize.com/post/getting-started-with-tmux/

**create deployment.yaml fast**
	
```
kubectl create deployment nginx --image=nginx --dry-run -oyaml > deploy_nginx.yaml
kubectl get deploy busybox --export -o yaml > exported.yaml
```

**static pods**

	https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
	https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-2-b4f5c636eb4
	
	```
	 ps auxfw | grep kubelet
	--config=/var/lib/kubelet/config.yaml
	staticPodPath: /etc/kubernetes/manifests
	```
**config maps**
	
	```kubectl create configmap app-config --from-literal=key123=value123```
  ```
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

	https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-3-2e7b44e89a3b
	```
	ETCDCTL_API=3 etcdctl help
	ETCDCTL_API=3 etcdctl — endpoints=[ENDPOINT] — cacert=[CA CERT] — cert=[ETCD SERVER CERT] — key=[ETCD SERVER KEY] snapshot save [BACKUP FILE NAME]
	kubectl describe pod etcd-master -n kube-system
	```
**kubeadm upgrade**



**rolling upgrade and rollback**
	https://kubernetes.io/docs/reference/kubectl/cheatsheet/ #Updating Resources
	
```
kubectl run nginx --image=nginx  --replicas=3
kubectl set image deploy/nginx nginx=nginx:1.9.1
kubectl rollout status deploy/nginx
kubectl rollout undo deploy/nginx
kubectl rollout status deploy/nginx
```
	
**taints and tolerations**
	https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

**affinity and anti-affinity**
	https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature
```
	kubectl label nodes sdombi-k8s-worker1 flavor=2u8g
```
```
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

**jsonpath exp**

**kube-controll**

**cheatsheet**
#run though all of these at least 3 times
https://kubernetes.io/docs/reference/kubectl/cheatsheet/ 


**labelselectors**

**create pv**
```
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
	https://linuxacademy.com/cp/courses/lesson/course/4019/lesson/6/module/327
	
```
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
```
kubectl create secret generic my-secret --from-literal=foo=bar -o yaml --dry-run > my-secret.yaml
kubectl create -f my-secret.yaml
```

```
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

```
	kubectl run kubeserve2 --image=chadmcrowell/kubeserve2
	kubectl expose deployment kubeserve2 --port 80 --target-port 8080 --type LoadBalancer

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

**kubectl explain**

**kubectl api-resources**
