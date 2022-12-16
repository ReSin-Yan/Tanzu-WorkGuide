Last update: 2022/12  
目前版本:1.3.3  
# Tanzu Application Platform  

## Tanzu Application Platform簡易介紹     
Tanzu Application Platform簡稱TAP，主要功能為提供營運團隊一個方便使用、可擴展性及安全的平台


VMware在2020/9月Java開發者大會SpringOne上，揭露Kubernetes應用開發營運（DevOps）平臺Tanzu Application Platform（TAP）的測試版，

Tanzu是VMware因應企業應用轉向容器化，而推出的開發及管理產品線。VMware指出，TAP是以Kubernetes為核心，融合其原有技術Spring Framework及早前的微服務平臺Tanzu Application Service的應用感知能力，提供開發工具及流程，以加速開發應用及服務到公有雲或本地部署經認證的Kubernetes叢集上。

VMware表示，Tanzu Application Service和最新的TAP差異在，TAP使用Tanzu及kubectl指令行介面（CLI）工具，而前者則使用VMware Tanzu Operations Manager及Cloud Foundry CLI。針對想轉移到TAP的Tanzu Application Service用戶，VMware也提供移轉和相容工具。


[參考網站](https://www.ithome.com.tw/news/148845 "link")  


## 安裝步驟   

### 環境簡介&安裝需求  

| 基本需求 | 用途or說明 | 
|-------|-------|
| VMWare Tanzu Network | 用來下載安裝需要之檔案，安裝必要之images(可以推送到私有倉庫) |
| container image registry |  Docker Hun or Harbor(本文使用Docker hun對外安裝的方式) |
| Gitlab or Github  | 以Repo為單位，進行配置workload |  
| 包含StorageClass的kubernetes  | 完整安裝需要10GB以上的可使用空間,以及設定成default |  
| DNS Record  | 服務使用Contour及enovy，可以等TAP建立完成後設定 |  
| Kubernetes需求  | v1.22,v1.23,v1.24, 本篇文章使用Tanzu(7.0 U3e or later) |  
| Kubernetes配置  | 16 vCPUs(total),24 GB of RAM(total),100 GB(pernode) |  
| 確認EULA  | 總共有三個網頁需要確認授權 |  


## Linux Client 準備  

環境更新及安裝基本套件  
```
sudo apt-get update && sudo apt-get -y upgrade
sudo apt-get -y install vim build-essential curl ssh
sudo apt-get install net-tools
```

安裝Docker engine(會由此台VM將Images推送至docker hub)    
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

確認安裝版本
```
sudo docker --version
```

安裝helm
```
cd
sudo apt-get install -y curl
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
``` 

安裝Kubectl工具(本篇文章透過原生Kubectl的安裝方式)  
[VMWare提供之安裝方式]([https://www.ithome.com.tw/news/148845](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) "link")  



## Kubernetes 操作及環境準備    

確認Kuberentes服務  
已下指令是Tanzu環境登入的指令  
如果是其它Kubernetes的平台，需要確認能夠正常的執行Kubernetes的相關操作  
可以跳轉到下一章節  

登入到Taznu環境(IP,路徑)  
```
export KUBECTL_VSPHERE_PASSWORD=P@ssw0rd
kubectl vsphere login --server=192.168.170.73 --insecure-skip-tls-verify  --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-name resin-tap-01  
kubectl config use-context resin-tap-01
```

下載gcallowroot yaml(TKC需要)  
```
sudo apt-get install -y git
cd 
git clone https://github.com/ReSin-Yan/NTUSTCourse
cd NTUSTCourse/Kubernetes
kubectl apply -f gcallowroot.yaml  
kubectl create clusterrolebinding default-tkg-admin-privileged-binding --clusterrole=psp:vmware-system-privileged --group=system:authenticated
```

安裝NFS sub-dir(如有Kubernetes本身已有StorageClass也建議設定)  

```
cd 
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=[ip] \
    --set nfs.path=/home/ubuntu/nfsshare
kubectl patch storageclass wcppolicy -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'    
```


## 安裝TAP  

install the Tanzu CLI 以及其插件(Tanzu指令的擴充包，可以輸入更多指令參數),從vmware network下載  
圖片待補  
```
tar -xvf tanzu-framework-linux-amd64.tar -C $HOME/tanzu
export TANZU_CLI_NO_INIT=true
cd $HOME/tanzu
export VERSION=v0.25.0
sudo install cli/core/$VERSION/tanzu-core-linux_amd64 /usr/local/bin/tanzu
tanzu version
tanzu plugin install --local cli all
tanzu plugin list 
```

Deploying Cluster Essentials v1.3,從vmware network下載  
圖片待補  
```
tar -xvf tanzu-cluster-essentials-linux-amd64-1.3.0.tgz
export INSTALL_BUNDLE=registry.tanzu.vmware.com/tanzu-cluster-essentials/cluster-essentials-bundle@sha256:54bf611711923dccd7c7f10603c846782b90644d48f1cb570b43a082d18e23b9
export INSTALL_REGISTRY_HOSTNAME=registry.tanzu.vmware.com
export INSTALL_REGISTRY_USERNAME=[VMWare Network Account]
export INSTALL_REGISTRY_PASSWORD=[VMWare Network Password]
./install.sh --yes
sudo cp $HOME/tanzu-cluster-essentials/kapp /usr/local/bin/kapp
sudo cp $HOME/tanzu-cluster-essentials/imgpkg /usr/local/bin/imgpkg
#install TAP(download from vmware registry)
export TAP_VERSION=1.3.3
kubectl create ns tap-install
```

添加secret  
```
tanzu secret registry add tap-registry \
--server registry.tanzu.vmware.com \
--username [VMWare Network Account] \
--password [VMWare Network Password]  \
--namespace tap-install \
--export-to-all-namespaces   \
--yes
```

從vmware官網將檔案list拉到本地端  
```
tanzu package repository add tanzu-tap-repository   \
--url $INSTALL_REGISTRY_HOSTNAME/tanzu-application-platform/tap-packages:$TAP_VERSION   \
--namespace tap-install
```

確認是否可以看到可安裝內容列表  
```
tanzu package available list --namespace tap-install
```

準備config(可以參考LINK檔案建立，根據環境設定配置檔案)  
```
tanzu package install tap -p tap.tanzu.vmware.com -v 1.3.3 --values-file  tap-values.yaml -n tap-install 
```
通常第一次安裝都會failed  
但是實際上並不是安裝失敗，而是images在下載，過了檢查時間  
可以過一小段時間再回頭檢查  


###   
確認安裝狀態  
```
tanzu package installed list -A
```
確認特定package status  
```
tanzu package installed  get tap-gui -n tap-install
```
刪除服務  
```
tanzu package installed delete tap -n tap-install
```
更新服務  
```
tanzu package installed update tap --values-file  tap-values.yaml -n tap-install
```


## 功能測試以及環境設定    

添加Images repo設定(需要跟tap-vauels設定相同，以下以docker為例)  
並且本設定會以demo的namespace當作工作環境  
```
kubectl create ns demo  
tanzu secret registry add registry-credentials \
  --server https://index.docker.io/v1/ \
  --username [Docker username] \
  --password [Docker password] \
  --namespace demo 
```

設定namespace的權限  
```
export TAP_DEV_NAMESPACE="demo"
curl -o serviceaccounts.yml https://raw.githubusercontent.com/benwilcock/TAPonLAP/main/TAPonLAPv1.1/serviceaccounts.yml
kubectl -n $TAP_DEV_NAMESPACE apply -f "serviceaccounts.yml"

cat <<EOF > rbac.yaml
apiVersion: v1
kind: Secret
metadata:
  name: tap-registry
  annotations:
    secretgen.carvel.dev/image-pull-secret: ""
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: e30K
---
apiVersion: v1
kind: Secret
metadata:
  name: git-ssh
  annotations:
    tekton.dev/git-0: github.com
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: 8J+UkQ==
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
secrets:
- name: registry-credentials
- name: git-ssh
imagePullSecrets:
- name: registry-credentials
- name: tap-registry
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default-permit-deliverable
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: deliverable
subjects:
  - kind: ServiceAccount
    name: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default-permit-workload
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: workload
subjects:
- kind: ServiceAccount
  name: default
EOF

kubectl -n demo apply -f rbac.yaml
```

測試部屬服務  
```
tanzu apps workload apply py-demo   --app py-demo   --git-repo https://github.com/jasonchiu17/tap-py-demo   --git-branch master   --type web   --annotation autoscaling.knative.dev/minScale=1   -n demo   -y
```


