# TKGm    

### 安裝Debug   

1.Linux Client必須事先安裝合符合以下條件
Docker  
Kubectl套件  
CPU大於4 Core  
RAM大於6 GB  


2.版本對應的OVF(範本)名稱必須完全符合  
不同版本OVF不相通  


3.kube-vip的IP網段需要跟DHCP的網段相同  
會造成management-cluster只會把Control-plane建立出來  
其他worker都不會建立  

4.如果在安裝過程中  
卡在bootstrape中的某一個步驟  
建議整台linux client刪掉重裝  
具體發生原因不明  



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

會顯示一串密碼之後  
4.使用密碼進行登入(workernodeIP從vsphere上看，並且無法直接連線，需要從client登入)  
```
ssh vmware-system-user@WorkerNodeIP  
``` 

[參考網站](https://sapphirelin.com/20210713-%E6%9C%80%E9%80%9F%E7%99%BB%E5%85%A5Tanzu-guest-cluster-node-%E6%96%B9%E6%B3%95-%E9%80%B2%E5%8E%BB-TKC-%E7%AF%80%E9%BB%9E/ "link")  

