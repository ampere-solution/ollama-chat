# ollama-chat
### Prepare and Deploy Single Node Microk8s (Ubuntu 22.04)
##### Update Ubuntu OS
$ sudo apt update && sudo apt upgrade -y

##### Install MicroK8s from snap on your system
$ sudo snap install microk8s --channel=1.31-strict/stable

##### Check the instance 'microk8s is running'
$ sudo microk8s status --wait-ready | head -n4

##### Add your user to the ‘microk8s’ group for unprivileged access on your system
$ sudo adduser $USER snap_microk8s

##### Alias kubectl so it interacts with MicroK8s by default on your system
$ sudo snap alias microk8s.kubectl kubectl

##### Create new group 
$ newgrp snap_microk8s
$ groups

##### Check the node status, run kubectl get node on your system
$ kubectl get node -owide
cloudfest-demo         Ready    <none>   98d     v1.31.5   192.168.1.10   <none>        Ubuntu Core 20   5.15.0-130-generic   containerd://1.6.28

##### Enable add-ons dns, hostpath-storage, ingress, dashboard
$ sudo microk8s enable hostpath-storage ingress
$ sudo microk8s enable dashboard

##### Check the status to see if dns, hostpath-storage, ingress, dashboard are enabled
$ sudo microk8s status

##### Show running pods
$ kubectl get pod -A
NAMESPACE     NAME                                         READY   STATUS    RESTARTS   AGE
ingress       nginx-ingress-microk8s-controller-2q7w4      1/1     Running   0          5m18s
kube-system   calico-kube-controllers-759cd8b574-pjcqp     1/1     Running   0          60m
kube-system   calico-node-pq9km                            1/1     Running   0          24m
kube-system   coredns-7896dbf49-fhq8v                      1/1     Running   0          60m
kube-system   dashboard-metrics-scraper-6b96ff7878-hdlnp   1/1     Running   0          3m27s
kube-system   hostpath-provisioner-5fbc49d86c-qbs2z        1/1     Running   0          5m19s
kube-system   kubernetes-dashboard-7d869bcd96-mzjfw        1/1     Running   0          3m27s
kube-system   metrics-server-d6f74bb9f-5f7s8               1/1     Running   0          3m28s

##### Create dashboard-ingress.yaml file to access Kubernetes dashboard
$ cat << EOF > dashboard-ingress.yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: ingress-kubernetes-dashboard
  namespace: kube-system
  generation: 1
  annotations:
    kubernetes.io/ingress.class: public
    nginx.ingress.kubernetes.io/backend-protocol: HTTPS
    nginx.ingress.kubernetes.io/configuration-snippet: |
      chunked_transfer_encoding off;
    nginx.ingress.kubernetes.io/proxy-body-size: '0'
    nginx.ingress.kubernetes.io/proxy-ssl-verify: 'off'
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/server-snippet: |
      client_max_body_size 0;
spec:
  rules:
    - host: <your dashboard dns name> 
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubernetes-dashboard
                port:
                  number: 443
EOF

##### Apply dashboard-ingress.yaml
$ kubectl apply -f dashboard-ingress.yaml 

##### Check to see if ingress is created and using the correct <dashboard dns name>
$ kubectl get ingress -A

##### Retrieve token to access the Kubernetes dashboard
$ sudo microk8s kubectl describe secret -n kube-system microk8s-dashboard-token


