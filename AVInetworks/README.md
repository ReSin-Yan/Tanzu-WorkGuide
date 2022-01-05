# AVInetworks相關問題  


### AKO Debug Note  

#### Labels does not match with cluster name for SE group
如果在安裝過程中出現AKO的pod不停的CrashBackLookOff  
透過以下指令看pod的log  
```
kubectl logs -n avi-system ako-0  
```
出現  
![img](https://github.com/ReSin-Yan/Tanzu-WorkGuide/blob/main/AVInetworks/Labelsdoesnotmatch.PNG)  


發生原因為  
每一組AVI的Service Engine group只能有一組AKO  
解決方法有二  

1.新建一組Service Engine group，並且再配置AKO的yaml  
修改serviceEngineGroupName  
新建Service Engine group輸入名子即可，並不需要額外設定  

2.再刪除原有AKO的時候，確實刪除掉AKO  
不能直接把cluster打掉，會讓設定卡在Service Engine group上面  
造成後面的服務上不去  
正確刪除方式  
```
kubectl edit configmap avi-k8s-config -n avi-system  
helm delete $(helm list -n avi-system -q) -n avi-system  
```
參考資料:  
[AKO 1.4 DebugNote](https://avinetworks.com/docs/ako/1.4/ako-faq/ "link")  
