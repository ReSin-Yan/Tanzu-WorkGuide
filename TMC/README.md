# TMC   

### 筆記   

1.所有符合CNCF條件的k8s都可以納入  
採用安裝agent的方式  
第一次需要對外  
之後只需要開proxy即可  


```
kubectl-vsphere login --vsphere-username administrator@vsphere.local --server=x.x.x.x --insecure-skip-tls-verify  
```
2.切換到supvisorCluster的namespace  
```
kubectl config use-context namespace-NAME  
```
3.取得系統管理員密碼  
```
kubectl get secret CLUSTER-NAME-ssh-password -o jsonpath='{.data.ssh-passwordkey}' | base64 -d; echo  
```
