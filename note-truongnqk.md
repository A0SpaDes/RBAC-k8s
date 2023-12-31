*** Để tạo 1 file kubeconfig cho 1 đối tượng client cần 2 giai đoạn chính:
1./ Tạo phương thức xác thực - authentication.
  + Xác định tên user, tên group.
  + tạo file client-certificate & client-key cho client (nói rõ ở bên dưới).
  + 3 bước add CA, cert, key vào file kubeconfig -> kubectl config --kubeconfig=<name> set-cluster / set-credentials / set-context
2./ Cấp quyền sau khi xác thực - authorization process.
  + tạo Role RBAC & RoleBinding -> cấp những quyền thao tác đến các resource cụ thể cho User/Group trên 1 NameSpace chỉ định.
  + tạo ClusterRole RBAC & ClusterRoleBinding -> cấp những quyền thao tác đến các resource cụ thể cho User/Group trên Cluster-Resources chỉ định.
  
*** Thực hiện:
1./ ví dụ về: tạo file client-certificate & client-key cho client

Nếu user đang ssh không phải root -> copy file CA.crt & CA.key (trong /etc/kubernetes.pki/) ra trước để map với gen crt & key riêng cho User:
Tạo new key cho user truongnqk
```
  openssl genrsa -out truongnqk.key 2048
```
Tạo file csr chứa thông tin name user và tên group - ví dụ dưới đây, tên user là "truongnqk admin" và group là "Blade-k8s-lab".
```
  openssl req -new -key truongnqk.key -out truongnqk.csr -subj "/CN=truongnqk admin/O=Blade-K8s-Lab"
```
Kết hợp với file ca.crt và ca.key của host để gen ra crt và key của user.
```
  openssl x509 -req -in truongnqk.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out truongnqk.crt -days 3650
```

2./ tạo Role RBAC & RoleBinding
# role-temp.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  #  namespace: blade-k8s-lab
  name: manage-admin
  #  labels:
  #    rbac.authorization.k8s.io/aggregate-to-cluster-admin: "true"
rules:
- apiGroups: ["", "apps", "batch", "networking.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
```

---
# rolebinding-temp.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: manage-admin
  namespace: default
subjects:
#- kind: User
#  name: "truongnqk admin"
- kind: Group
  name: Blade-K8s-Lab  
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: manage-admin
  apiGroup: rbac.authorization.k8s.io
```
---
# cluster-role-temp.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-role-manage
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
```
---
# cluster-rolebinding-temp.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-binding-manage
subjects:
- kind: Group
  name: Blade-K8s-Lab
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-role-manage
  apiGroup: rbac.authorization.k8s.io
```  
---
