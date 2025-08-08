
* https://minikube.sigs.k8s.io/docs/handbook/
* https://minikube.sigs.k8s.io/docs/handbook/controls/
* https://minikube.sigs.k8s.io/docs/start/
* 
* 


```
kubectl get service --namespace kube-system
kubectl port-forward --namespace kube-system service/registry 5000:80
```
Now check out - http://localhost:5000/v2/_catalog

```
 1002  brew tap hashicorp/tap
 1004  brew install hashicorp/tap/terraform
 1005  brew update
 1006  brew upgrade
 1007  kubectl version --client
 1008  brew install kubectl
 1009  kubectl version --client
 1010  cd /u/src
 1011  cd k8s
 1012  mkdir k8s
 1013  cd k8s
 1014  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
 1015  kubectl cluster-info
 1016  kubectl version
 1017  hombrew update kubectl
 1018  brew update kubectl
 1019  brew upgrade kubectl
 1020  kubectl
 1021  kubectl --version
 1022  kubectl version
 1023  which kubectl
 1024   kubectl version --client --output=yaml
 1025  kubectl cluster-info
 1026  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl-convert"
 1027  kubectl convert --help
 1028  minikube start
 1029  brew install minikube
 1030  brew unlink minikube
 1031  brew link minikube
 1032  minikube start
 1033  kubectl convert --help
 1034  kubectl get po -A
 1035  minikube kubectl -- get po -A
 1036  alias kubectl="minikube kubectl --"
 1037  minikube dashboard
 1039  kubectl create deployment hello-minikube --image=docker.io/nginx:1.23
 1040  kubectl expose deployment hello-minikube --type=NodePort --port=80
 1041  kubectl get services hello-minikube
 1042  kubectl port-forward service/hello-minikube 7080:80
 1044  kubectl create deployment balanced --image=docker.io/nginx:1.23kubectl expose deployment balanced --type=LoadBalancer --port=80\n
 1045  kubectl get services balanced
 1046  minikube pause
 1047  minikube unpause
 1048  kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
 1049  kubectl create deployment take2 --image=k8s.gcr.io/echoserver:1.4
 1050  kubectl expose deployment take2 --type=NodePort --port=8080
 1051  minikube service take2
 1052  h

  999  ifconfig -a| grep inet
 1000  brew tap hashicorp/tap
 1001  minikube tunnel

```

