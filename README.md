# cka
CKA exam preparation

static pods 
	https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
	https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-2-b4f5c636eb4
	 ps auxfw | grep kubelet
	--config=/var/lib/kubelet/config.yaml
	staticPodPath: /etc/kubernetes/manifests


etcd backup
	https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-3-2e7b44e89a3b
	ETCDCTL_API=3 etcdctl help
	ETCDCTL_API=3 etcdctl — endpoints=[ENDPOINT] — cacert=[CA CERT] — cert=[ETCD SERVER CERT] — key=[ETCD SERVER KEY] snapshot save [BACKUP FILE NAME]
	kubectl describe pod etcd-master -n kube-system
kubeadm upgrade



rolling upgrade and rollback
	https://kubernetes.io/docs/reference/kubectl/cheatsheet/ #Updating Resources
taints and tolerations
	https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

affinity and anti-affinity
	https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature
	kubectl label nodes sdombi-k8s-worker1 flavor=2u8g
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
	

jsonpath exp

kube-controll

cheatsheet

labelselectors

create pv
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

securityContext
	https://linuxacademy.com/cp/courses/lesson/course/4019/lesson/6/module/327

create service and ingress
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

kubectl explain

kubectl api-resources
