# ARGOCD - HUB & SPOKE Design
### List the contexts of the clusters
$ kubectl config get-contexts
CURRENT   NAME                                                 CLUSTER                               AUTHINFO                                             NAMESPACE
          user@hub-cluster.us-east-1.eksctl.io       hub-cluster.us-east-1.eksctl.io       user@hub-cluster.us-east-1.eksctl.io       
*         user@spoke-cluster-1.us-east-1.eksctl.io   spoke-cluster-1.us-east-1.eksctl.io   user@spoke-cluster-1.us-east-1.eksctl.io  
 $ kubectl config current-context
user@hub-cluster.us-east-1.eksctl.io
$ kubectl config use-context user@hub-cluster.us-east-1.eksctl.io
Switched to context "user@hub-cluster.us-east-1.eksctl.io".


https://argo-cd.readthedocs.io/en/stable/getting_started/
Install Argo CD:
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

$ kubectl get all -A

LIst the config maps of ago cd
kubectl get cm -n argocd
Edit argocd-cmd-params-cm  to make this insecure, http
kubectl edit cm argocd-cmd-params-cm  -n argocd


https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/argocd-cmd-params-cm.yaml
kubeclt edit configmap argocd-cmd-params-cm -n argocd

data:
  server.insecure: "true"

Chnage the type of argocd-server service to nodeport
 k edit svc argocd-server -n argocd
  type: NodePort
Take the ip adress of one of the EC2 machine and make sure the SG is updated with the port:
![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/97b332ad-e429-4bbe-acc7-abf2b08acb06)
 $ k get secrets -n argocd
NAME                          TYPE     DATA   AGE
argocd-initial-admin-secret   Opaque   1      38m
argocd-notifications-secret   Opaque   0      38m
argocd-secret                 Opaque   5      38m
gitpod /workspace/argocd-hub-spoke (main) $ k edit secrets argocd-initial-admin-secret -n argocd
Edit cancelled, no changes made.
gitpod /workspace/argocd-hub-spoke (main) $ k get secrets argocd-initial-admin-secret -n argocd -o yaml
apiVersion: v1
data:
  password: YkFvUUVNMHItMXNTU3dnag==
kind: Secret
metadata:
  creationTimestamp: "2024-01-11T01:20:23Z"
  name: argocd-initial-admin-secret
  namespace: argocd
  resourceVersion: "5496"
  uid: fd381325-679a-49c3-b269-992e94d2c2d8
type: Opaque
gitpod /workspace/argocd-hub-spoke (main) $ echo argocd-initial-admin-secret -n argocd | base64 --decode
j�(qbase64: invalid input
gitpod /workspace/argocd-hub-spoke (main) $ echo argocd-initial-admin-secret -n argocd^C base64 --decode
gitpod /workspace/argocd-hub-spoke (main) $ echo YkFvUUVNMHItMXNTU3dnag== base64 --decode
bash: ech: command not found
gitpod /workspace/argocd-hub-spoke (main) $ echo hi
hi
gitpod /workspace/argocd-hub-spoke (main) $ echo YkFvUUVNMHItMXNTU3dnag== | base64 --decode
bAoQEM0r-1sSSwgj

![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/3b54cb15-4e60-4c98-8709-1127b586ddc6)

Add cluster
![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/faa63ed0-1f40-44bb-9660-62eb0814f309)

gitpod /workspace/argocd-hub-spoke (main) $ argocd login 3.33.233.51:31400
WARNING: server certificate had error: tls: failed to verify certificate: x509: cannot validate certificate for 3.81.233.51 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context '3.81.233.51:31400' updated
gitpod /workspace/argocd-hub-spoke (main) $ kubectl config get-contexts
CURRENT   NAME                                                 CLUSTER                               AUTHINFO                                             NAMESPACE
*        user@hub-cluster.us-east-1.eksctl.io       hub-cluster.us-east-1.eksctl.io       user@hub-cluster.us-east-1.eksctl.io       
          user@spoke-cluster-1.us-east-1.eksctl.io   spoke-cluster-1.us-east-1.eksctl.io   user@spoke-cluster-1.us-east-1.eksctl.io   
gitpod /workspace/argocd-hub-spoke (main) $ argocd cluster add user@spoke-cluster-1.us-east-1.eksctl.io server 3.81.233.51:31400
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `user@spoke-cluster-1.us-east-1.eksctl.io` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0003] ServiceAccount "argocd-manager" created in namespace "kube-system" 
INFO[0003] ClusterRole "argocd-manager-role" created    
INFO[0003] ClusterRoleBinding "argocd-manager-role-binding" created 
INFO[0008] Created bearer token secret for ServiceAccount "argocd-manager" 
Cluster 'https://463ED5F24BF0FF928739B9588A2258354.gr7.us-east-1.eks.amazonaws.com' added

![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/7d2334c6-dbef-4a04-be9a-47dcf839f91e)

Deploying the application to EKS
configure application in argocd

![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/c1234bbc-5532-47c5-bf0d-f69aa88dcf91)

![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/46e24d66-3865-4258-8728-85082eab19c1)
![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/66f2bd2a-2669-4a2f-a454-20650727d427)
![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/74e69186-aabe-4678-a4f1-9d9b29a0084f)

![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/f4fe30ab-9167-4cd7-9fec-b32cd1e76fcd)

gitpod /workspace/argocd-hub-spoke (main) $ kubectl config get-contexts
CURRENT   NAME                                                 CLUSTER                               AUTHINFO                                             NAMESPACE
*         user@hub-cluster.us-east-1.eksctl.io       hub-cluster.us-east-1.eksctl.io       user@hub-cluster.us-east-1.eksctl.io       
          user@spoke-cluster-1.us-east-1.eksctl.io   spoke-cluster-1.us-east-1.eksctl.io   user@spoke-cluster-1.us-east-1.eksctl.io   
gitpod /workspace/argocd-hub-spoke (main) $ kubectl config use-context user@spoke-cluster-1.us-east-1.eksctl.io
Switched to context "user@spoke-cluster-1.us-east-1.eksctl.io".
gitpod /workspace/argocd-hub-spoke (main) $ kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/guestbook-ui-6b7f6d9874-94252   1/1     Running   0          4m53s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/guestbook-ui   ClusterIP   10.100.183.4   <none>        80/TCP    4m54s
service/kubernetes     ClusterIP   10.100.0.1     <none>        443/TCP   94m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook-ui   1/1     1            1           4m55s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/guestbook-ui-6b7f6d9874   1         1         1       4m55s
gitpod /workspace/argocd-hub-spoke (main) $ 

Change the following file and sync he application in argocd:
![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/cfeb302b-f408-4dba-be68-ba71278780a9)

![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/9d7f89ce-ede9-4f42-bdec-c3849c419565)

gitpod /workspace/argocd-hub-spoke (main) $ kubectl get cm
NAME               DATA   AGE
guest-book         1      18s
kube-root-ca.crt   1      109m
gitpod /workspace/argocd-hub-spoke (main) $ kubectl delete configmap guest-book
configmap "guest-book" deleted
![image](https://github.com/eapenm/argocd-hub-spoke/assets/13297994/53aca16a-d0e0-49a5-96cc-d076970d27c2)

Sync the application using the argocd UI - COnfig map applied automatically by argocd

gitpod /workspace/argocd-hub-spoke (main) $ kubectl get cm
NAME               DATA   AGE
guest-book         1      28s
kube-root-ca.crt   1      110m



### Delete the Cluster using below:
eksctl delete cluster --name hub-cluster --region us-east-1

eksctl delete cluster --name spoke-cluster-1 --region us-east-1

