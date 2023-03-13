Last update: 2023/03  
目前版本:1.4.1  
# Tanzu Application Platform  

## Tanzu Application Platform簡易介紹     
Tanzu Application Platform簡稱TAP，主要功能為提供營運團隊一個方便使用、可擴展性及安全的平台


VMware在2020/9月Java開發者大會SpringOne上，揭露Kubernetes應用開發營運（DevOps）平臺Tanzu Application Platform（TAP）的測試版，

Tanzu是VMware因應企業應用轉向容器化，而推出的開發及管理產品線。VMware指出，TAP是以Kubernetes為核心，融合其原有技術Spring Framework及早前的微服務平臺Tanzu Application Service的應用感知能力，提供開發工具及流程，以加速開發應用及服務到公有雲或本地部署經認證的Kubernetes叢集上。

VMware表示，Tanzu Application Service和最新的TAP差異在，TAP使用Tanzu及kubectl指令行介面（CLI）工具，而前者則使用VMware Tanzu Operations Manager及Cloud Foundry CLI。針對想轉移到TAP的Tanzu Application Service用戶，VMware也提供移轉和相容工具。


[參考網站](https://www.ithome.com.tw/news/148845 "link")  

以下內容針對再Air-Gap環境當中進行安裝設定  

## 安裝步驟   

### 環境簡介&安裝需求  

| 基本需求 | 用途or說明 | 
|-------|-------|
| VMWare Tanzu Network | 用來下載安裝需要之檔案，安裝必要之images(推送到私有倉庫) |
| 包含StorageClass的kubernetes  | 完整安裝需要10GB以上的可使用空間,以及設定成default |  
| DNS Record  | 服務使用Contour及enovy，可以等TAP建立完成後設定 |  
| Kubernetes需求  | v1.23,v1.24, 本篇文章使用Tanzu(7.0 U3e or later) |  
| Kubernetes配置  | 16 vCPUs(total),24 GB of RAM(total),100 GB(pernode) |  
| 確認EULA  | 總共有三個網頁需要確認授權 |  
| NFS  | 總共有三個網頁需要確認授權 |  

### 離線環境需要額外準備的內容  

| 基本需求 | 用途or說明 | 安裝連結 |
|-------|-------|-------|
| Harbor |  用來存放images，其中包含了TAP安裝所需要的images，以及在suppluchain中產生的images |
| Gitlab or Github  | 以Repo為單位，進行配置workload |  
| nexus server  | supply chain當中的Test code的步驟中，會需要用到的maven package |  
| nginx server  | supply chain當中的Scan code/images 的步驟中，會需要用到的grype db |  

Tips: 其中的nexus server以及nginx server，都是採用 opensource 的安裝方式，如果有習慣的可以使用其他安裝方式  
Tips2: Harbor需要先放置nginx1.16,maven,nfs-subdir-external-provisioner 這三個images  
Tips3: 由於測試內容使用NFS當作儲存空間(storageclass)，所以額外準備nfs-subdir-external-provisioner的images，沒有則不必  

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
[VMWare提供之安裝方式](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ "link")  

## Kubernetes 操作及環境準備    

確認Kuberentes服務  
已下指令是Tanzu環境登入的指令  
如果是其它Kubernetes的平台，需要確認能夠正常的執行Kubernetes的相關操作  

登入到Taznu環境(IP,路徑)  
```
export KUBECTL_VSPHERE_PASSWORD=[VSPHERE_PASSWORD]
kubectl vsphere login --server=192.168.170.73 --insecure-skip-tls-verify  --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-name resin-tap-01  
kubectl config use-context resin-tap-01
```

部屬gcallowroot yaml(TKC需要)  
```
kubectl apply -f gcallowroot.yaml  
kubectl create clusterrolebinding default-tkg-admin-privileged-binding --clusterrole=psp:vmware-system-privileged --group=system:authenticated
```

## 準備Harbor,Gitlab憑證    

```

```

## 安裝NFS作為storageclass   

```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=10.8.7.199 --set nfs.path=/home/ubuntu/nfsshare
kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
手動修改nfs的deploy，其中的images

## 安裝tanzucli   

[下載連結](https://network.tanzu.vmware.com/products/tanzu-application-platform#/releases/1250091)  
下載版本v0.25.4.4
```
cd tanzu
mkdir tanzucli
tar -xvf tanzu-framework-linux-amd64-v0.25.4.4.tar	

sudo install cli/core/v0.25.4/tanzu-core-linux_amd64 /usr/local/bin/tanzu
sudo install cli/core/v0.25.4/tanzu-core-linux_amd64 /usr/local/bin/tanzu
tanzu version
tanzu plugin install --local cli all
```

## 安裝tanzu cluster ess(1.4)  

需要先從registry.tanzu.vmware.com將檔案下載並推送至harbor已進行離線安裝  

```
cd 
cd tanzu
mkdir tanzuclusterress
#Download from https://network.tanzu.vmware.com/products/tanzu-cluster-essentials/
tar -xvf tanzu-cluster-essentials-linux-amd64-1.4.1.tgz
sudo cp kapp /usr/local/bin/kapp
sudo cp imgpkg /usr/local/bin/imgpkg
```

編輯install.sh 在其中的一行檔案加入--registry-ca-cert-path(需事先準備harbor的憑證，範例路徑為/home/ubuntu/token/ca.crt)  
如下  
```
./imgpkg pull -b $INSTALL_BUNDLE -o ./bundle/ --registry-ca-cert-path /home/ubuntu/token/ca.crt
```

之後執行  
```
kubectl create namespace kapp-controller
kubectl create secret generic kapp-controller-config --namespace kapp-controller --from-file caCerts=/home/ubuntu/token/ca.crt
./install.sh --yes
```

會成功安裝以下兩個元件至k8s  
kapp-controller  
secretgen-controller  

## 安裝TAP1.4.1  


```
export TAP_VERSION=1.4.1
kubectl create ns tap-install
sudo usermod -aG docker ubuntu
```

```
export IMGPKG_REGISTRY_HOSTNAME=nvharbor.zeronetap.lab
export IMGPKG_REGISTRY_USERNAME=admin
export IMGPKG_REGISTRY_PASSWORD=Harbor12345
export TAP_VERSION=1.4.1
export REGISTRY_CA_PATH=/home/ubuntu/token/ca.crt
```

```
tanzu secret registry add tap-registry --server   $IMGPKG_REGISTRY_HOSTNAME --username $IMGPKG_REGISTRY_USERNAME --password $IMGPKG_REGISTRY_PASSWORD --namespace tap-install  --export-to-all-namespaces --yes
tanzu package repository add tanzu-tap-repository   --url $IMGPKG_REGISTRY_HOSTNAME/tap-packages/tap:$TAP_VERSION   --namespace tap-install

tanzu package repository get tanzu-tap-repository --namespace tap-install
tanzu package available list --namespace tap-install
```

準備好harbor的憑證，透過以下指令產出憑證轉譯  
```
export CA_CERT=$(sed -e ':X; N; $!bX; s/\n/\\n/g; s/$/\\n/' < YOURHARBORCA.CRT)
echo $CA_CERT
```

接著在profile內新增以下資訊(放在shared block)  
```
shared:
  ca_cert_data: "-----BEGIN CERTIFICATE-----\xxxxxxxxxxxxxxxxxxxxxxxxx\n-----END CERTIFICATE-----\n"
  image_registry:
    project_path: "harbor/tap-packages/tap"
    username: "admin"
    password: "Harbor12345"
```

執行安裝  
```
tanzu package install tap -p tap.tanzu.vmware.com -v 1.4.1 --values-file  tap-values.yaml -n tap-install
```

## 安裝TBS1.9.4  

如果需要執行完整版本的TAP(包含scan、code test)  
需要在額外部屬TBS  

```
tanzu package available list buildservice.tanzu.vmware.com --namespace tap-install
tanzu package repository add tbs-full-deps-repository   --url $IMGPKG_REGISTRY_HOSTNAME/tap-packages/tbs-full-deps:1.9.4 --namespace tap-install
tanzu package install full-tbs-deps -p full-tbs-deps.tanzu.vmware.com -v 1.9.4 -n tap-install
```


