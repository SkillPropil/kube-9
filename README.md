# kube-9
## Задание 1 Создайте конфигурацию для подключения пользователя
1) Создал сертификат
```
openssl genrsa -out skillpropil 2048
openssl x509 -req -in skillpropil.csr -CA /var/snap/microk8s/7232/certs/ca.crt -CAkey /var/snap/microk8s/7232/certs/ca.key -CAcreateserial -out skillpropil.crt -days 500
openssl req -new -key skillpropil.key -out skillpropil.csr -subj "/CN=skillpropil/O=pyanitsa"
Certificate request self-signature ok
subject=CN = skillpropil, O = pyanitsa
```
2) Настройте конфигурационный файл kubectl для подключения.
```
kubectl config set-credentials skillpropil --client-certificate=cert/skillpropil.crt --client-key=cert/skillpropil.key
User "skillpropil" set

kubectl config set-context skillpropil-context --cluster=microk8s-cluster --user=skillpropil
Context "skillpropil-context" created.

[root@skillpropilserv-01 kube-9-local]# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:16443
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: skillpropil
  name: awertoss-context
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
- context:
    cluster: microk8s-cluster
    user: skillpropil
  name: skillpropil-context
current-context: microk8s
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: awertoss
  user:
    client-certificate: /root/kube-9-local/cert/skillpropil.crt
    client-key: /root/kube-9-local/cert/skillpropil.key
- name: skillpropil
  user:
    client-certificate: /root/kube-9-local/cert/skillpropil.crt
    client-key: /root/kube-9-local/cert/skillpropil.key
```
3) Создайте роли и все необходимые настройки для пользователя.
```
[root@skillpropilserv-01 kube-9]# kubectl apply -f role.yaml 
role.rbac.authorization.k8s.io/pod-describe-logs created
[root@skillpropilserv-01 kube-9]# kubectl apply -f role_map.yaml 
rolebinding.rbac.authorization.k8s.io/pod-reader created
```
4) Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (kubectl logs pod <pod_id>, kubectl describe pod <pod_id>).
```
kubectl get role pod-desc-logs -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"Role","metadata":{"annotations":{},"name":"pod-desc-logs","namespace":"default"},"rules":[{"apiGroups":[""],"resources":["pods","pods/log"],"verbs":["watch","list","get"]}]}
  creationTimestamp: "2023-08-14T06:17:18Z"
  name: pod-desc-logs
  namespace: default
  resourceVersion: "5233519"
  uid: 0298e69a-700d-405d-a009-f58d0ea6d873
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - pods/log
  verbs:
  - watch
  - list
  - get
```
5) Манифесты [role.yaml](role.yaml) и [role_map.yamp](role_map.yaml)
