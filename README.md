# k8s-test

```
---------------------------------------------------------------------------------------------------------
||    k8s     ||         kubernetes      ||        master        ||        worker        || 
---------------------------------------------------------------------------------------------------------
*********************************************************************************************************
**            **      ****************               *************************               
**          **      *                  *             *************************
**        **        *                  *             ******      
**      **          *                  *             ******     
**    **             *                *              ******
**  **                *              *               ******      
*****                 ****************               *************************
**  **                *               *              *************************
**   **              *                 *                            **********           
**    **            *                  *                            **********
**     **           *                  *                            **********
**      **          *                  *                            **********
**       **          *                *              *************************
**        **          ****************               *************************   

```
-----------------------------------------------------------------------------------------------------------------
### StorageClass

- Specifications: -
	* If we will not write storageClassName in PVC manifests file then by default storage class will assign to it.
	* To check the storage class has default assign capabilities or not
	* annotations:
	       storageclass.kubernetes.io/is-default-class=true
	* This annotation marks made them a default storage class.
	* If we will assign storageClassName in manifests file then PVC get volume assigned by given storageClassName.
	* If we write wrong storageClassName in the manifests file then PVC status will be 'pending' and reason marked by 'ProvisioningFailed'.
	* To allocate dynamic provisioning we have to create a Storage Class, assign them a name, provisioner, parameters etc. After that when ever it will get requests from cluster user, It will create dynamically volume. Cluster Administrator don't need to create PV manifests file for this.
	* Note: Cluster user has to write storageClassName in PVC manifests file otherwise if not write storageClassName then it will get default storage class. 

-----------------------------------------------------------------------------------------------------------------
- 
	* If you want to bind the PVC to pre-provisioned PV then you have write the empty string after the storageClassName.
	* storageClassName: ""
	* This things bound to allocate the default storage class volume.
	* Explicitly set storageClassName to "" if you want the PVC to use a pre-provisioned PersistentVolume.

-----------------------------------------------------------------------------------------------------------------

### ConfigMaps

- Specifications: -
	* Key must contain an alphanumeric, hypens(-), dots(.), underscores(_). 
	* We cannot give a key name for a directory path.
	* kubectl create cm thanos --from-file=data=data/      ---> wrong
- A whole directory
	* kubectl create cm thanos --from-file=data/           ---> correct
- Data stored in a file    
	* kubectl create cm data --from-file=config.json       ---> correct
- Key/Value 
	* kubectl create cm data --from-file=key=value         ---> correct
- A file stored in a custom key
	* kubectl create cm data --from-file=strike=data.yaml  ---> correct
- You can also mark a reference to a ConfigMap as optional (by setting configMapKeyRef.optional: true). In that 
  case, the container starts even if the ConfigMap doesn’t exist.
  * Environment variables doesn't support hyphens(-) is in the key. `http-top` is not a valid identifier. It should be http_top.

----------------------------------------------------------------------------------------------------------------

### OS Upgrades

- If any Pod is schedule on Node and that is not operate by ReplicaSets, Deployments, Replicationcontroller then it will not possible to unschedule in normal way. 
  * One thing, we can do first delete the Pod then process ahead for unschedule to Node.
  * Second thing, run the command with forcefully it will delete single Pods and proceed unschedule process.

- We have to run the following command: -

```
kubectl drain node02 --force --ignore-daemonsets

```
> Note: - That Pod is not a part of any self heal resources so it will be deleted for ever and not recoverable or not scheduleable on other Nodes.

- To get back the unschedule Node to on Ready/Scheduling state then run the following command: -

``` 
kubectl uncordon node02

```
If you wants to Node be unscheduleable and not ready state without evicted already available Pod then run the following command: -

```
kubectl cordon node01

```
- To get back on Schedule/Ready state then run the following command: -

```
kubectl uncordon node01

```

### ETCD BACKUP & RESTORE

- ETCD Backup: -

- ETCD cluster volume already mounted in master node at location /var/lib/etcd. If you will inspect/describe etcd-master Pod in kube-system namespace.
  You will see in volumeMounts.mountPath to volumes.hostPath.

- To maintain a backup of ETCD cluster: - 
	* We have to follow these steps.
  * ETCDCTL_API=3 because functions are working in API version 3.

> Note: - You cannot perform ETCD API version 2 commands, You have to change it into API version 2 to make it work.
  
  * You can do export ETCDCTL_API=3 to expose in system ENV.

```
  etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=https://127.0.0.1:2379  <path of location to save snapshot of ETCD>

```
- Before attempting this, check it that following command is working or not.

```
etcdctl member list --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=https://127.0.0.1:2379

O/P :  7b20d9b132c379ac, started, controlplane, https://172.17.0.8:2380, https://172.17.0.8:2379 

```
> That means successfully ran.

- You can check version of ETCD and location of Keys and Certificates from etcd.yaml manifests file or To inspect/describe the etcd-master Pod,
  which is located in kube-system namespace.

- To check the version of ETCD inside etcd Pod.

> List of Pods in kube-system namespaces

```
kubectl get po -n kube-system

# To check the version of ETCD cluster

kubectl exec etcd-master -n kube-system -- etcdctl version

```
- You can check version of etcd and other important location of etcd from static etcd manifests file. (/etc/kubernetes/manifests/etcd.yaml)
            or
 	You can inspect/describe etcd-master Pod which is located in kube-system namespaces.

```
kubectl describe pod etcd-master -n kube-system

```
- You will get all the details of etcd cluster.

- ETCD Restore: -

- To restore backup in ETCD cluster
  * First `export ETCDCTL_API=3` in system ENV.
  * To take a help of etcdctl subcommands: -

```
etcdctl snapshot restore -h

```
  * To take a restore of data with help of backup data.
```
# Make sure to do `export ETCDCTL_API=3` otherwise add in beginning of the command.

ETCDCTL_API="3" etcdctl snapshot restore \
--cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key \
--endpoints=127.0.0.1:2379 --name master \
--data-dir="/var/lib/etcd-backup" --initial-advertise-peer-urls="https://127.0.0.1:2380" \
--initial-cluster-token="etcd-cluster-1" \
--initial-cluster="master=https://127.0.0.1:2380"   /tmp/snapshot-data.db

```
```
Options: -
> --name                                --> Name of the controlplane node component
> --data-dir                            --> New location of etcd directory
> --initial-cluster-token               --> New name of ETCD cluster 
> --initial-advertise-peer-urls         --> Interact with master node and helpful while restoring data
> --initial-cluster                     --> Initial cluster configuration for restore bootstrap

```
- After this setup configuration, move to the static Pod manifest path of kubernetes. (/etc/kubernetes/manifests)
- Add these lines into `command` section of etcd.yaml manifest file.
  * --data-dir=/var/lib/etcd-backup
  * --initial-cluster-token=etcd-cluster-1
- Change into the volumes section: -
  * volumeMounts.mountPath: - /var/lib/etcd to /var/lib/etcd-backup
  * volumes.hostPath: - /var/lib/etcd to /var/lib/etcd-backup
- After this whole configuration of ETCD cluster, kubelet will detect these changes and automatically restart the Pods. It will take a few minutes. 

### E2E Test-0
- Warnings: -
```
2020/07/11 03:50:19 Warning: Couldn't find directory src/k8s.io/kubernetes under any of GOPATH /root/go, defaulting to /root/go/src/k8s.io/kubernetes
2020/07/11 03:50:19 Warning: Couldn't find directory src/sigs.k8s.io/cloud-provider-azure under any of GOPATH /root/go, defaulting to /root/go/src/k8s.io/cloud-provider-azure
2020/07/11 03:50:19 Warning: Couldn't find directory src/k8s.io/kubernetes under any of GOPATH /root/go, defaulting to /root/go/src/k8s.io/kubernetes
```
----
1. You can make a directory of above missing directories in under $HOME/go/src to resolve this issues.
2. You can ignore above errors. It will not impact in your e2e test.
3. Clone this repo in your current working directory (https://github.com/kubernetes/test-infra.git).
```
 $ cd $HOME
 $ git clone https://github.com/kubernetes/test-infra.git
```
4. After successfully cloned. Move into the directory test-infra and run the following command: -
> It will downloaded some modules/dependencies/packages.
```
 $ cd /root/test-infra/kubetest
 $ GO111MODULE=on go install ./kubetest
```
5. Completion of downloading and extraction of modules/packages. Move into the /root/go/bin path.
```
 $ cd /root/go/
 $ ls
 $ bin  pkg
 $ cd bin/
 $ ls
 $ kubetest
```
6. Set a environment variable in your system to allocate identity of about master component.
```
# Example : "172.17.0.51:6443"
$ export KUBE_MASTER_API="{master-ip}:{master-port}"
# Example : master, controlplane
$ export KUBE_MASTER={master-name}
```
7. After successfully setup all configurations. Run the following command to perform e2e test.
```
# Move into the bin directory of GOPATH (/root/go/bin).
$ cd /root/go/bin/

# kubetest is available in an executable mode. It will take long time to run the tests.
$ kubetest --test --provider=skeleton --extract=v1.18.0 --test_args=--ginkgo.focus="\[Conformance\]" > test-result

Options: -
1. --test        - This flag tells kubetest to run the test.e2e binary built/extracted from the kubernetes/kubernetes repo.
2. --provider    - This flag tells to perform in which platform. "skeleton" for local environment.
3. --extract     - This flag tells what version of kubernetes would like to test. Must be match with your Kubernetes cluster version.   
```
> Note: - `--extract` value must be match with your `Kubernetes version`. 

8. To run the selected resources. Run the following command: -
```
$ cd /root/go/bin
$ kubetest --test --provider=skeleton --extract=v1.18.0 --test_args=--ginkgo.focus="\[Secrets\]" > secrets-testresult

# Deployments | Pods | ConfigMaps | Secrets | ServiceAccount | RoleBinding | Role | ClusterRole | ClusterRoleBinding | more..
```
9. You can check the details about kubernetes version, success, failed status from stored file.
```
$ cat secrets-testresult
```

### E2E Test-1 

- Correct way: -

1. Set a environment variable in your system to allocate identity of about master component.
```
# Example : "172.17.0.51:6443"
# You can check kube-apiserver running details from `kubectl cluster-info`

$ export KUBE_MASTER_API="{master-ip}:{master-port}"

# Example : master, controlplane

$ export KUBE_MASTER={master-name}
```

2. Clone a repo in your current working directory (https://github.com/kubernetes/test-infra.git).
```
$ git clone https://github.com/kubernetes/test-infra.git
```

3. Move into the cloned directory test-infra and run the following command: -
```
$ cd /root/test-infra

# Run this from the clone repo

$ GO111MODULE=on go install ./kubetest
```

4. (Optional) Check/List the directory contents: -
```
$ ls -l /root/go
total 12
drwxr-xr-x 4 root root 4096 Jul 11 05:04 bin
drwxr-xr-x 4 root root 4096 Jun  8 22:04 pkg
drwxr-xr-x 8 root root 4096 Jul 11 04:53 src

$ cd /root/go

$ ls -l bin/
total 99832
-rwxr-xr-x 1 root root 41330568 Jun  8 22:04 kind
-rwxr-xr-x 1 root root 60892683 Jul 11 04:56 kubetest
```

5. Move into the bin directory of GOPATH (/root/go/bin). `kubetest` is available in executable mode.
```
# Move into the bin directory of GOPATH (/root/go/bin).
$ cd /root/go/bin/

# kubetest is available in an executable mode. It will take long time to run the tests.

$ kubetest --test --provider=skeleton --extract=v1.18.0 --test_args=--ginkgo.focus="\[Secrets\]" > secret-testresult

Options: -
1. --test        - This flag tells kubetest to run the test.e2e binary built/extracted from the kubernetes/kubernetes repo.
2. --provider    - This flag tells to perform in which platform. "skeleton" for local environment.
3. --extract     - This flag tells what version of kubernetes would like to test. Must be match with your Kubernetes cluster version.   
```
> Note: - `--extract` value must be match with your `Kubernetes version`. 

6. To run the selected resources. Run the following command: -
```
$ cd /root/go/bin

$ kubetest --test --provider=skeleton --extract=v1.18.0 --test_args=--ginkgo.focus="\[ConfigMaps\]" > cm-testresult

# Deployments | Pods | ConfigMaps | Secrets | ServiceAccount | RoleBinding | Role | ClusterRole | ClusterRoleBinding | more..
```

7. You can check the details about kubernetes version, success, failed status from stored file.
```
$ cat cm-testresult
```

8. You can add `--timeout` flag to finish a test in a given duration time but not all.
```
$ cd /root/go/bin/

# This "Conformance" test for 10min. Otherwise it will take 1hour+ to successfully complete.
# Testing timing is 10 minutes.

$ kubetest --test --provider=skeleton --extract=v1.18.0 --test_args=--ginkgo.focus="\[Conformance\]" --timeout=10m > quick_testresult 
```

