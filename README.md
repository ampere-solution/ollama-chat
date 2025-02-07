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

##### Apply dashboard-ingress.yaml
Note:  replace "your dashboard dns name" with your dns
$ kubectl apply -f dashboard-ingress.yaml 

##### Check to see if ingress is created and using the correct <dashboard dns name>
$ kubectl get ingress -A

##### Retrieve token to access the Kubernetes dashboard
$ sudo microk8s kubectl describe secret -n kube-system microk8s-dashboard-token


