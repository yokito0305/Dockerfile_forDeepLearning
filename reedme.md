# Dockerfile 預先安裝各種 DeepLearning 套件合併 jupyterlab 與 ssh 環境

* 可逐步建立映像檔，基本環境搭建大約 10-15GB
* 或是從 deep-learning-with-jupyter 資料夾中一鍵 build 完成，但會使用大約40GB流量(過程預估1小時)。
* 需要時請自行參考原始碼做調整

---

## 目錄

- [Dockerfile 預先安裝各種 DeepLearning 套件合併 jupyterlab 與 ssh 環境](#dockerfile-預先安裝各種-deeplearning-套件合併-jupyterlab-與-ssh-環境)
  - [目錄](#目錄)
    - [第一種方法 : 一鍵建立映像檔](#第一種方法--一鍵建立映像檔)
      - [1. 移動至 deep-learning-with-jupyter 資料夾並執行 build](#1-移動至-deep-learning-with-jupyter-資料夾並執行-build)
      - [2. 建立 container](#2-建立-container)
      - [3. 啟動密碼為使用者名稱，輸入後會自動在後台開啟 ssh 與 jupyterlab server](#3-啟動密碼為使用者名稱輸入後會自動在後台開啟-ssh-與-jupyterlab-server)
    - [第二種方法 : 逐步建立映像檔](#第二種方法--逐步建立映像檔)
      - [1. 移動至 base-setting 資料夾並執行 build](#1-移動至-base-setting-資料夾並執行-build)
      - [2. 移動至 miniconda-setup 資料夾建立 conda 環境映像檔](#2-移動至-miniconda-setup-資料夾建立-conda-環境映像檔)
      - [2. 移動至 jupyterlab-setup 資料夾建立 conda 與 jupyterlab 環境](#2-移動至-jupyterlab-setup-資料夾建立-conda-與-jupyterlab-環境)
      - [3. 移動至 ssh-jupyterlab-setup 資料夾建立 ssh 環境](#3-移動至-ssh-jupyterlab-setup-資料夾建立-ssh-環境)
      - [4. 移動至 deep-learning-package-setup 資料夾安裝 deep-learning 套件](#4-移動至-deep-learning-package-setup-資料夾安裝-deep-learning-套件)


---
### 第一種方法 : 一鍵建立映像檔

#### 1. 移動至 deep-learning-with-jupyter 資料夾並執行 build
```bash=
cd ./deep-learning-with-jupyter
docker build -t deep-learning-with-jupyter . --no-cache --network=host

# docker build -t deep-learning-with-jupyter . --build-arg username=user --build-arg userUID=1000 --build-arg userGID=100 --no-cache --network=host
```

* **\-\-build\-arg 為可選項**
    * username : 可定義使用者名稱，**後續 root 與 jupyterlab 預設密碼皆為使用者名稱**
    * userUID, userGID : 使用者與群組 ID 設定

#### 2. 建立 container
```bash=
docker run -it --gpus all -p 8888:8888 -p 22:22 -v D:\Users\AppData\Local\ubuntu\testimage:/workspace --name deep-learning-with-jupyter deep-learning-with-jupyter
```
* __docker start 容器名稱__ //啟動容器
* __docker restart 容器名稱__  //重新啟動容器
* __docker stop 容器名稱__ //關閉容器
* __docker run \-it \-\-gpus all \-p 自己網路port:容器網路port 	 \-v 自己的路徑:容器中的路徑 \-\-name=名子 映像檔名:tag__(tag沒指定的話預設為最新) //run 起一個作業環境
    * \-it (跟容器裡的終端連線)
    * \-\-gpus all (將gpu控制權給容器)
    * \-p (自己的網路port對應容器得網路port(需要再填))
    * \-v (與容器共用資料夾空間(容器刪除後不會消失))
    * \-\-name (為容器取名子)
    * \-\-privileged=true (啟動容器真正的 root 權限，開啟後才能修改網卡、容器中啟動容器等等操作)

* 其他用法 :
    * __docker exec \-it 容器名稱 bash__ //與容器裡的中連線
    * __docker rm 容器名稱__ //刪除容器(要先stop容器)
    * __docker rmi 映像檔名__ //刪除映像檔
    * __docker images__ //列出以下載的映像檔
    * __docker ps__ //列出以執行的容器
    * __docker ps \-a__ //列出全部容器包含未啟動
    * __docker rename 現在容器名稱 要取的容器名稱__

#### 3. 啟動密碼為使用者名稱，輸入後會自動在後台開啟 ssh 與 jupyterlab server
* 登入 jupyterlab 方法 : 打開瀏覽器，網址列輸入 **localhost:8888/lab**
* ssh 連入 : 打開終端，輸入 ssh [username]@localhost -p 22
* ssh 可參考 : [使用 SSH 連入VMware-ubuntu](https://hackmd.io/@yokito/SSHtoVMware)

* 裡面已經預先建立一個 python 虛擬環境 **(name=jupyterlab, python=3.12)**
```bash=
conda activate jupyterlab # 啟動 python 虛擬環境
```

---

### 第二種方法 : 逐步建立映像檔

#### 1. 移動至 base-setting 資料夾並執行 build
```bash=
cd ./base-setting
docker build -t yokito/base-setting . --network=host
```

#### 2. 移動至 miniconda-setup 資料夾建立 conda 環境映像檔
```bash=
cd ../miniconda-setup
docker build -t yokito/miniconda-setup . --network=host

# docker build -t yokito/miniconda-setup . --build-arg username=user --build-arg userUID=1000 --build-arg userGID=100 --build-arg USER_HOME_PATH=/home/$username --network=host
```
* **\-\-build\-arg 為可選項**
    * username : 可定義使用者名稱，**後續 root 與 jupyterlab 預設密碼皆為使用者名稱**
    * userUID, userGID : 使用者與群組 ID 設定
    * USER_HOME_PATH : 為使用者主目錄 (\$cd ~)

#### 2. 移動至 jupyterlab-setup 資料夾建立 conda 與 jupyterlab 環境
```bash=
cd ../jupyterlab-setup
docker build -t yokito/jupyterlab-setup . --network=host
```

#### 3. 移動至 ssh-jupyterlab-setup 資料夾建立 ssh 環境
```bash=
cd ../ssh-jupyterlab-setup
docker build -t yokito/ssh-jupyterlab-setup . --network=host
```

* ***到這步大約使用 10-15GB 流量與空間，如果不需要先安裝 deep-learning 套件在這邊就可以停手了***

使用方法 : 
```bash=
docker run -it --rm -p 8888:8888 -p 22:22 --gpus all --name ssh-setup yokito/ssh-jupyterlab-setup
```

#### 4. 移動至 deep-learning-package-setup 資料夾安裝 deep-learning 套件
```bash=
cd ../deep-learning-package-setup
docker build -t deep-learning-jupyterlab . --network=host
```

* 映像檔製作完成後，就能透過映像檔建立容器使用了

使用方法 : 
```bash=
docker run -it --rm -p 8888:8888 -p 22:22 --gpus all --name deep-learning-jupyterlab deep-learning-jupyterlab
```
* [其他使用方法](#3-啟動後密碼為使用者名稱輸入後會成功開啟-ssh-與-jupyterlab-server)