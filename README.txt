

#pull the needed docker iamges
docker pull mysql
docker pull alpine
docker pull woahbase/alpine-mysql


#prereq - start the cluster with a valid CNI
minikube delete
minikube start --cni=cilium
kubectl create -f https://raw.githubusercontent.com/cilium/cilium/1.5.6/examples/kubernetes/1.15/cilium-minikube.yaml
kubectl get pods -n kube-system -l k8s-app=cilium


#create 2 namespaces to work from
kubectl create ns developer
kubectl label namespace/developer purpose=developer
kubectl create ns infrastructure
kubectl label namespace/infrastructure purpose=infrastructure


#deploy mysql into the infrastructure namespace
kubectl -n infrastructure apply -f application/mysql/mysql-deployment.yaml


#deploy pod1 and pod2 to the developer namespace
kubectl -n developer apply -f application/pod/pod1-deployment.yaml
kubectl -n developer apply -f application/pod/pod2-deployment.yaml

#both pods from developer ns can mysql to db pod on infrastructure

#some useful resources
URL: https://github.com/ahmetb/kubernetes-network-policy-recipes


#lock infrastructure ns down with netpol default deny all
kubectl -n infrastructure apply -f network/default-deny-all.yaml


#allow pods from developer ns with the label pod1 to access the db
kubectl -n infrastructure apply -f network/netpol-allow-developer-pod1.yaml


