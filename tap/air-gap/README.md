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
| Harbor |  用來存放images，其中包含了TAP安裝所需要的images，以及在suppluchain中產生的images |
| Gitlab or Github  | 以Repo為單位，進行配置workload |  
| 包含StorageClass的kubernetes  | 完整安裝需要10GB以上的可使用空間,以及設定成default |  
| DNS Record  | 服務使用Contour及enovy，可以等TAP建立完成後設定 |  
| Kubernetes需求  | v1.23,v1.24, 本篇文章使用Tanzu(7.0 U3e or later) |  
| Kubernetes配置  | 16 vCPUs(total),24 GB of RAM(total),100 GB(pernode) |  
| 確認EULA  | 總共有三個網頁需要確認授權 |  

### 離線環境需要額外準備的內容  


