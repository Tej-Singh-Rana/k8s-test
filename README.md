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
	* Before attempting this, check it that following command is working or not.

```
etcdctl member list --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=https://127.0.0.1:2379

O/P :  7b20d9b132c379ac, started, controlplane, https://172.17.0.8:2380, https://172.17.0.8:2379 

```
	* That means successfully run.

- You can check version of ETCD and location of Keys and Certificates from etcd.yaml manifests file or To inspect/describe the etcd-master Pod,
  which is located in kube-system namespace.

- To check the version of ETCD inside etcd Pod.
> List of Pods in kube-system namespaces

```
kubectl get po -n kube-system

kubectl exec etcd-master -n kube-system -- etcdctl version

```
- You can check version of etcd and other important location of etcd from static etcd manifests file. (/etc/kubernetes/manifests/etcd.yaml)
            or
 	You can inspect/describe etcd-master Pod which is located in kube-system namespaces.

```
kubectl describe pod etcd-master -n kube-system

```
- You will get all the details of etcd cluster.