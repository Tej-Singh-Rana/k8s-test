## Role based access control (RBAC) policies in Kubernetes

### Private Key
```
 openssl genrsa -out tej.key 2048
```

### Certificate Signing Request 
```
openssl req -new -key tej.key -out tej.csr -subj "/CN=tej"
```

### Certificate
```
openssl x509 -req -in tej.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out tej.crt -days 365
``` 

### To check the details of Certificate
```
openssl x509 -in tej.crt -text -noout
```

### To set a new cluster 
```
kubectl config set-cluster cloud --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --server=https://${MASTER_IP}:6443
```
### To create a new set-credentials | To create user
```
kubectl config set-credentials tej@kubernetes --client-certificate="/root/tej.crt" --client-key="/root/tej.key" --embed-certs=true
```

### To create a new set-context
```
kubectl config set-context tej@kubernetes --cluster=kubernetes --user=tej@kubernetes
```

### To set a new context
```
kubectl config use-context tej@kubernetes
```

