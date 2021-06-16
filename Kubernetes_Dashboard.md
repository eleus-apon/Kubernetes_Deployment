# Accessing the Dashboard

Step 15: Deploying the Dashboard UI
The Dashboard UI is not deployed by default. To deploy it, run the following command:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

Verify Status:
Now we will check the Dashboard's creation and deployment status using this command.
kubectl get deployments -n kubernetes-dashboard

Create Modules:
Next, we will create two modules; one for the dashboard and one for the metrics. The dash n (-n) flag represents a namespace.
kubectl get pods -n kubernetes-dashboard


Check Service:
Now we can check the NodePort service that we modified earlier. Notice the kubectl get command now defines 'services,' which includes the nodeport IP's.
kubectl get services -n kubernetes-dashboard 

Dashboard v2  the v2 dashboard is deployed in its own namespace kubernetes-dashboard:
$$ kubectl -n kubernetes-dashboard describe deployments kubernetes-dashboard


Change parameters on the command line
The v1.x dashboard is usually deployed in the kube-system namespace:

$ kubectl -n kube-system get deployments



To edit Kuberenetes-Dashborad:
$ kubectl -n kubernetes-dashboard describe deployments kubernetes-dashboard

URL: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy

\\\\
Step16: Creating a Service Account
We are creating Service Account with name admin-user in namespace kubernetes-dashboard first.

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF


Creating a ClusterRoleBinding
In most cases after provisioning cluster using kops, kubeadm or any other popular tool, the ClusterRole cluster-admin already exists in the cluster. We can use it and create only ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.

cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

Getting a Bearer Token
Now we need to find token we can use to log in. Execute following command:

kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"

Output:
eyJhbGciOiJSUzI1NiIsImtpZCI6ImZDbU1mTFVyd0RjNkdrMjNWMUhGNmdFdjd6aHRDNnN2Ql96NGxoMFB2X1UifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWRsbDY5Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1ODc4MWFjZS1iNmFjLTQyNmYtYjMwNy1kNzk1YTJjZmVmNzQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z_ZF6p6rQ-7bWUYg-qCGp9-q-pgjJEtVqdsXcN_BpK1PcVJN1DEMHNiaIOndKQh8U5Ge7_Pa5_osLFiyW9pIcfXXeRbwXfT39ZgeN2nDEj9_gbMP1VfDOliRipaQMzKmtVyanb780JwVhBAEz4Wtxri3J9FXPcfZGCeSOU8WF3CUSs5eymv7I_hicddXvv_b5su5wwdWZ4tBIy7WAh6y5OAcGjviuLlcj8FG8CLE8SRrP_E8KtFT9B5k_IjYbefkSrnnKAdpqhCQYrqbWx0nE3A01BEEe6UUsljqmqtFCOmeB7Hn-Wz3zAA6uzxRqWjz2_-t1nFXXv3dNfAyLsFIPg

////

Step: 16: Accessing the Dashboard UI

Command line proxy
You can access Dashboard using the kubectl command-line tool by running the following command:

kubectl proxy

If doesn't work, try below command.
kubectl proxy --address 0.0.0.0 --accept-hosts '.*'


URL: http://192.168.11.51:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy


If get an error as below:
The connection to the server localhost:8080 was refused - did you specify the right host or port?

Which means  Kubectl command did not able to find its config file.
 To sort it out.

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
