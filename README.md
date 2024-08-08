## Docker Desktop Issues

1. 到 Docker Desktop -> 點左側 Containers tab -> 選擇一個 container -> 點擊 Files 會看到 "Refreshing container files..." forever ，相關 issue https://github.com/docker/for-win/issues/14005 <br/>Solutions: <br/>
   1. https://stackoverflow.com/questions/62217678/can-i-roll-back-to-a-previous-version-of-docker-desktop 降低 Docker Desktop 的版本
   2. https://docs.docker.com/desktop/release-notes/#4261 下載 4.26.1 的版本
   3. 安裝 4.26.1 的版本取代原來的版本

---

## Docker - pull image, containerize, interactive with container

`docker pull ubuntu` will pull the latest version of ubuntu image <br/>

`docker run -it ubuntu` will create a container of ubuntu image and can interactive with container. -it means interactive <br/>

可以執行 `ls`, `cd`, `mkdir`, `touch` 等指令看看加上 -it option 是不是可以跟 container 互動，`exit` 可以退出 <br/>

如果忘了在 `docker run` 的時候 `-it` 可以先用 `docker ps` 找到正在運行的容器的 ID ，再使用 `docker exec` 命令進入容器，假設容器 ID 是 abc123 那進入容器的命令就是 `docker exec -it abc123 /bin/bash`

---

## Docker - create our own image

see /hello-docker <br/>

`cd hello-docker/` <br/>

`docker build -t hello-docker .` : `docker build` 用來 build image 。 `-t` t stands for tag 表示我們要給這個 image 一個標籤。 `.` 要指向 Dockerfile 所在的目錄，一個點表示當前目錄。 這個指令的完整意思就是使用當前目錄下的 Dockerfile 來 build 一個 Docker image，並且把這個 image 貼上 hello-docker 的標籤。 <br/>

`docker images` list docker images <br/>

`docker run hello-docker` will containerize hello-docker image ，然後會看到終端機輸出 "Hello Docker!" 。 <br/>

到 Docker Desktop -> 點左側 Containers tab -> 選擇一個 container -> 點擊 Files ，會看到 /app 下面有 Dockerfile 和 hello.js 兩個檔案，這就是我們從本地機器複製過去的 <br/>

### 從 shell mode 開啟我們的 application

1. 從 shell mode 進去，退出 Shell 同時停止容器
   ```sh
   docker run -it hello-docker sh # 在運行 `docker run` 命令時指定了一個命令，例如 `sh`，這個命令會覆蓋 Dockerfile 中定義的 CMD 指令 。 執行 `sh` 命令會啟動 shell 也就是交互式命令行界面
   node hello.js # 用 node run time 跑 hello.js
   exit # 退出 Shell 並退出容器，由於容器是通過 sh Shell 啟動的，當 sh Shell 退出時，整個容器也會停止運行。
   ```
2. 從 shell mode 進去，退出 Shell 但是不停止容器

   如果你希望容器繼續運行，但只退出 sh Shell，可以考慮啟動容器時在後台運行它（不使用 -it），然後通過 docker exec 進入容器。這樣即使退出了 Shell，容器也會繼續運行。例如：

   在後台啟動容器：

   ```sh
   docker run -d --name my-container hello-docker
   ```

   進入正在運行的容器：

   ```sh
   docker exec -it my-container sh
   ```

   退出 sh Shell：

   ```sh
   exit
   ```

   這樣即使你退出了 sh Shell，容器 my-container 也會繼續運行。

### CMD 指令執行完畢會退出容器嗎

在 Docker 容器中執行完 CMD 指令後，容器的行為取決於 CMD 指令的內容以及它的運行結果。如果 CMD 指令執行完畢並退出，容器也會隨之退出。具體來說：

1.  短暫執行的 CMD 指令：
    如果 CMD 指令是執行一個短暫的任務（例如打印一條消息），那麼這個任務完成後，容器就會退出。例如：

    ```Dockerfile
    Copy code
    FROM ubuntu
    CMD ["echo", "Hello, World!"]
    ```

    當你運行這個映像時：

    ```sh
    docker run my-echo-image
    ```

    容器會打印 Hello, World!，然後立即退出。

2.  長期運行的 CMD 指令：
    如果 CMD 指令是啟動一個長期運行的進程（例如一個服務器或一個持續運行的腳本），那麼容器會保持運行，直到該進程結束。例如：

    ```Dockerfile
    FROM ubuntu
    CMD ["sleep", "3600"]
    ```

    當你運行這個容器時，它會運行 sleep 3600 命令，並在一小時後停止：

    ```sh
    docker build -t long-lived-container .
    docker run long-lived-container
    ```

總結來說，Docker 容器在 CMD 指令執行完畢後會根據指令的性質來決定是否退出。如果 CMD 指令是短暫執行的，容器會立即退出；如果是長期運行的，容器會保持運行，直到該進程結束。

---

## Docker - React Demo

### Create React Porject

`npm create vite@latest react-docker`

1. npm create：這是用於創建新項目的命令。
2. vite@latest：指定使用最新版本的 Vite。Vite 是一個前端開發構建工具。
3. react-docker：指定項目創建在 ./react-docker 目錄下，並且項目名稱是 react-docker。
4. 命令執行完後會津入交互式介面，選擇 React 回車 -> TypeScript 回車

<br/>

`cd react-docker` 移動到 react-docker 目錄 <br/>

在 react-docker/ 下建立 Dockerfile 檔案，實際代碼請參考 react-docker/Dockerfile <br/>

### react-docker/Dockerfile 的筆記

#### 如何創建新的用戶和群組（之後要用這個 user 執行程序）

`addgroup app`和`adduser -S -G app app`這兩個命令在 Unix/Linux 系統中使用，主要是為了創建一個新的用戶和一個新的群組，並將該用戶加入該群組。

1. `addgroup app`:
   `addgroup` 是用來創建一個新的群組。
   app 是新群組的名稱。
   所以這個命令會創建一個名為 app 的群組。
2. `adduser -S -G app app`:
   `adduser` 是用來創建一個新的用戶。
   -S 參數表示創建一個系統用戶，這個用戶通常是沒有登錄權限的，並且用來運行系統服務或守護進程。
   -G app 參數指定新用戶將會被添加到名為 app 的群組中。
   最後的 app 是新用戶的名稱。
   所以這個命令會創建一個名為 app 的系統用戶，並且將該用戶添加到名為 app 的群組中。

這兩個命令是用來創建一個新的群組和一個新的系統用戶，並將該用戶加入該群組。這在設置服務或應用程式時很常見，可以用來管理權限和隔離不同的服務。

#### 為什麼不用 root user 執行程序，要為了執行程序創建新的用戶和群組？

創建新的群組和用戶，而不是直接使用 root，有重要的安全方面的原因：

安全性：

1. 最小權限原則：這是一個重要的安全原則，意味著應該只給予程序或用戶執行其任務所需的最小權限。使用 root 用戶運行程序意味著該程序擁有整個系統的最高權限，如果該程序被攻擊者利用，整個系統都可能被攻擊。
2. 減少風險：如果某個應用程序運行在一個特定的用戶下，並且該應用程序出現漏洞，攻擊者只能取得該用戶的權限，而不能取得整個系統的控制權。

#### 在 Dockerfile 中 USER 指令和 RUN su 是一樣的嗎？

RUN su app 和 USER app 是兩個不同的指令，它們的作用和適用場景也有所不同。讓我們來詳細比較這兩個指令。

USER app <br/>
USER app 指令設置了接下來所有 Dockerfile 指令（包括 CMD 和 ENTRYPOINT）將以指定的用戶身份運行。這是一個全局設置，並且會持續到 Dockerfile 結束。

示例：

```Dockerfile
FROM ubuntu

# 創建用戶 app
RUN useradd -m app

# 切換到用戶 app
USER app

# 設置工作目錄
WORKDIR /home/app

# 複製文件到工作目錄
COPY . .

# 安裝應用依賴（以 app 用戶身份運行）
RUN npm install

# 設置容器啟動命令（以 app 用戶身份運行）
CMD ["node", "app.js"]
```

在這個例子中，從 USER app 指令開始，後續所有的指令都是以 app 用戶身份運行的。

---

`RUN su app` <br/>

`RUN su app `指令是用來在單個 RUN 指令內切換用戶身份的。這通常用於臨時切換用戶身份來執行某些命令。需要注意的是，這種方法並不會影響 Dockerfile 中後續的指令，它只在當前的 RUN 指令中有效。

示例：

```Dockerfile
FROM ubuntu

# 創建用戶 app
RUN useradd -m app

# 切換到用戶 app 並運行命令
RUN su app -c "echo 'Hello from app user!'"

# 後續指令仍然以 root 用戶身份運行
WORKDIR /root

# 複製文件到工作目錄
COPY . .

# 安裝應用依賴（以 root 用戶身份運行）
RUN npm install

# 設置容器啟動命令（以 root 用戶身份運行）
CMD ["node", "app.js"]
```

在這個例子中，只有 RUN su app -c "echo 'Hello from app user!'" 這條指令是以 app 用戶身份運行的，後續的指令依然是以 root 用戶身份運行。 <br/>
-c 是 su 命令的一個選項，用來指定要執行的命令。在使用 su 命令切換用戶時，可以通過 -c 選項來執行特定的命令並在命令執行完畢後立即退出。

比較與選擇 <br/>
持續效果：如果希望從某個點開始，後續所有的指令都以非特權用戶運行，應使用 USER app。 <br/>
臨時效果：如果只希望某一條 RUN 指令以非特權用戶運行，可以使用 RUN su app -c "command"。

#### WORKDIR /app 和 RUN cd /app 一樣嗎？

在 Dockerfile 中，WORKDIR /app 和 RUN cd /app 是不一樣的，儘管它們都會改變當前的工作目錄。這裡有一些關鍵的區別：

1.  持久性：

    WORKDIR /app 設置了一個持久的工作目錄。當你設置了 WORKDIR，後續的所有指令（RUN、CMD、ENTRYPOINT、COPY、ADD）都將在這個目錄中執行，直到再次改變工作目錄。
    RUN cd /app 僅在該 RUN 指令執行期間改變工作目錄。一旦該指令執行完畢，工作目錄會重置回之前的狀態。

2.  簡潔性：

        WORKDIR /app 使得 Dockerfile 更加簡潔和易讀，因為你只需設置一次工作目錄，後續指令會自動在這個目錄中執行。
        使用 RUN cd /app 則需要在每一個需要使用該目錄的指令中重複這個操作，這會使 Dockerfile 顯得冗長。

    最佳實踐：

WORKDIR /app 是更推薦的做法，因為它符合 Dockerfile 的最佳實踐，提供更清晰的意圖和結構。
例如：

```dockerfile
FROM node:14

# 使用 WORKDIR 設置工作目錄
WORKDIR /app

# 複製文件到工作目錄
COPY . .

# 安裝依賴
RUN npm install

# 啟動應用
CMD ["npm", "start"]
```

相比之下，如果用 RUN cd /app，會是這樣：

```dockerfile
FROM node:14

# 複製文件到根目錄
COPY . /app

# 切換到 /app 目錄並安裝依賴
RUN cd /app && npm install

# 切換到 /app 目錄並啟動應用
CMD ["sh", "-c", "cd /app && npm start"]
```

可以看到，使用 WORKDIR 更加簡潔和易於維護。

### Build React Docker Image And Run It, Set Docker Network, Set Docker Volume

#### Build React Docker Image And Run It

在 react-docker/ 下建立 .dockerignore 檔案，寫入 node_modules/ 。 <br/>

因為我們會在 Dockerfile 用 `RUN npm install` 指令，而 `npm install` 就會根據 package.json 和 package-lock.json 安裝 dependencies，所以可以忽略 node_modules/ <br/>

`docker build -t react-docker .` build docker image <br/>

`docker run react-docker` use react-docker image to create a docker container<br/>

用瀏覽器訪問 http://localhost:5173/ ，會發現連不上，錯誤訊息：無法連上這個網站 localhost 拒絕連線。ERR_CONNECTION_REFUSED。 <br/>

#### Set Docker Network

這是因為雖然我們在 Dockerfile 用了 `EXPOSE 5173` 命令，但是 `EXPOSE 5173` 指的是通知 Docker 這個 container 要去聽 5173 port ，但是 host 也就是本地機器卻不知道去聽 5173 port 。 container 是 run 在 isolated environments ，所以不會 expose port to host machine 或其他人， container 外是連不進 container 裡面的。 <br/>

要讓 host machine 知道 Docker container 在聽 5173 port 需要用到 Docker 的 port mapping ， port mapping 可以讓我們在 Docker container 和 host machine 之間 map ports <br/>

`docker run -p 5173:5173 react-docker` -p 5173:5173: 這部分指定端口映射。格式為 主機端口:容器端口。這意味著將主機上的 5173 端口映射到容器內的 5173 端口，從而可以通過 http://localhost:5173 訪問容器內的應用程式。 <br/>

連到 http://localhost:5173 會發現還是沒通，看終端機會顯示

```sh
  VITE v5.3.4  ready in 134 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
```

是 vite 的問題，在 vite 也要 expose port，他要我們在 react-docker/package.json 修改 dev 命令，在命令後面加上 --host

```json
  "scripts": {
    "dev": "vite --host"
  }
```

修改完我們需要重 build image ，我們可以先通過 Docker Desktop 刪除剛剛那些 images and containers ，再下 `docker build -t react-docker .` build docker image ，再下 `docker run -p 5173:5173 react-docker` ： 叫 Docker 運行一個 container ，並把 host machine 的 5173 port map 到 container 的 5173 port ，指定用 react-docker image 啟動 container ，訪問 http://localhost:5173/ 這次就會成功連進 container 了 <br/>

查看一下終端機會發現輸出變成

```sh
> vite --host


  VITE v5.3.4  ready in 180 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: http://172.17.0.2:5173/
```

這個輸出顯示了 Vite 應用程序在 Docker 容器內部成功啟動，並且正在監聽兩個地址：

Local: http://localhost:5173/
這個地址是指容器內部的 localhost，並不是主機的 localhost。這是因為容器內部的網絡環境是隔離的。

Network: http://172.17.0.2:5173/
這是容器在 Docker 的內部網絡中的 IP 地址。容器內部的應用程序可以通過這個地址訪問，但主機上的瀏覽器無法直接訪問這個地址。

我們到 react-docker/src/App.tsx 把裡面的一行代碼 `<h1>Vite + React</h1>` 替換成 `<h1>Docker is awesome!</h1>` ，回到瀏覽器看卻發現沒有改變，這是因為你改了代碼但是 image 和 container 並沒有改變，到 Docker Desktop -> 左側欄 Containers Tab -> 點及我們剛剛啟動的 container -> 點擊 Files Tab -> 找到 /app/src/App.tsx 右鍵 Edit file 會發現裡面的代碼還是 `<h1>Vite + React</h1>` ，如果我們想要看到剛剛我們做的新的更改就必須重新 build image and run it ，這很麻煩，這時候就可以用 Volume 解決這個問題 <br/>

### Set Docker Volumn

#### What is Volume

在 Docker 中，卷（volume） 是一種持久化數據的機制，用於存儲和共享容器中的數據。卷可以在容器之間共享，並且可以持久化存儲，即使容器刪除後數據也不會丟失。這使得卷在容器化應用程序中非常重要，特別是在需要持久化數據的場景下，例如數據庫存儲、配置文件、日誌文件等。

##### 為什麼使用卷？

1. 數據持久化：卷允許數據在容器重新啟動或刪除後仍然存在。
2. 數據共享：卷可以在多個容器之間共享數據。
3. 數據分離：將應用數據與應用程序代碼分離，增強安全性和數據管理。
4. 性能：卷通常比將數據存儲在容器的聯合文件系統（union file system）中具有更好的性能。

##### 如何使用卷？

使用卷的常見方法有以下幾種：

1. 創建和使用卷
   你可以先創建一個卷，然後在運行容器時使用這個卷。

   ```bash
   # 創建一個卷
   docker volume create my_volume

   # 使用這個卷運行容器
   docker run -d -v my_volume:/app --name my_container my_image
   ```

   這樣，卷 my_volume 中的數據將被掛載到容器內的 /app 目錄。

2. 匿名卷
   在運行容器時可以直接創建匿名卷：

   ```bash
   docker run -d -v /app --name my_container my_image
   ```

   這會創建一個匿名卷並掛載到容器內的 /app 目錄。

3. 具名卷
   具名卷允許你指定一個具名卷並將其掛載到容器內的某個目錄：

   ```bash
   docker run -d -v my_named_volume:/app --name my_container my_image
   ```

4. 綁定掛載（Bind Mount）
   你也可以將本地主機上的目錄或文件掛載到容器內：

   ```bash
   docker run -d -v /path/on/host:/app --name my_container my_image
   ```

   這會將本地主機上的 /path/on/host 目錄掛載到容器內的 /app 目錄。

##### 示例

假設你有一個 React 應用並希望在容器中運行，同時保持依賴包的隔離：

```bash
docker run -p 5137:5137 -v "${pwd}:/app" -v /app/node_modules react-docker
```

這裡使用了兩個卷：

1. -v "${pwd}:/app"：將本地當前目錄映射到容器內的 /app 目錄。這樣你可以在本地編輯代碼，並在容器內運行應用。
2. -v /app/node_modules：創建一個匿名卷並掛載到容器內的 /app/node_modules 目錄。這樣可以避免本地的 node_modules 覆蓋容器內的依賴包。

##### 總結

卷是 Docker 中用於數據持久化和共享的一種機制。通過使用卷，你可以確保數據在容器的生命周期之外持續存在，並可以在多個容器之間共享數據。這對於構建可靠且可擴展的容器化應用程序非常重要。

#### Bind Mount 綁定掛載

ref: https://docs.docker.com/reference/cli/docker/container/run/#volume

剛剛提到如果每次改 code 都要 rebuild image and run container 會很痛苦，這時就可以通過 Volume 的 Bind Mount 將本地主機上的目錄或文件掛載到容器內，

要解決這個問題可以把主機目錄掛載到 Docker container 裡的目錄，這樣在更改主機目錄內的東西時會即時映射到被掛載的 Docker container 中的目錄，要告訴 Docker mount current working directory 到 Docker container 裡的 /app 可以在 `docker run` 指令加上 `-v "$(pwd):/app` ，完整的指令是 `docker run -p 5173:5173 -v "${pwd}:/app" react-docker` 。

執行後會出現錯誤 `sh: vite: not found` ，這是因為我們沒有主機目錄沒有安裝 dependencies ，我們在主機目錄下執行 `npm i` 後，再 `docker run -p 5173:5173 -v "${pwd}:/app" react-docker` 啟動一次 docker container 會出現錯誤 `Error: Cannot find module @rollup/rollup-linux-arm64-musl.` ，

這是因為 Bind Mount 會把本地的 node_modules 目錄覆蓋容器內的 node_modules 目錄，而本地的依賴包版本可能與容器內需要的不一致，特別是在跨平台（如不同的操作系統或架構）時。像我的主機是 MAC OS 但是 Docker image 是 alpine linux ，直接把 MAC OS 的依賴包掛載到 alpine linux 中是會出問題的。

這時候就需要使用匿名卷或具名卷保護容器內的 node_modules 目錄。

#### Anonymous Volume 匿名卷

匿名卷（anonymous volume）是在你運行容器時，Docker 自動創建的未命名的卷。這些卷不會與主機上的任何特定目錄或文件綁定，但它們可以防止覆蓋容器內的目錄。匿名卷通常用於確保某些容器內的數據不會被本地主機上的數據覆蓋。

##### 匿名卷的使用場景

匿名卷特別有用於以下場景：

保護容器內的依賴包：例如，防止本地的 node_modules 目錄覆蓋容器內的 node_modules 目錄。
防止覆蓋容器內的配置文件或數據：確保某些容器內的數據或配置文件不會被本地的文件覆蓋。

##### 如何使用匿名卷

在 Docker 運行命令中，匿名卷的語法如下：

```bash
docker run -p 5137:5137 -v "${pwd}:/app" -v /app/node_modules react-docker
```

在這個命令中，`-v /app/node_modules` 創建了一個匿名卷並將其掛載到容器內的 /app/node_modules 目錄。

##### 具體示例

假設你有一個 React 應用，並且你的項目目錄結構如下：

```sh
my-app/
├── node_modules/
├── src/
├── package.json
└── Dockerfile
```

當你運行以下命令時：

```bash
docker run -p 5137:5137 -v "${pwd}:/app" -v /app/node_modules react-docker
```

這裡發生的事情如下：

1. `-v "${pwd}:/app"`：將本地當前目錄（${pwd}）映射到容器內的 /app 目錄。這樣你可以在本地編輯代碼，並在容器內查看變更。
2. `-v /app/node_modules`：創建一個匿名卷並掛載到容器內的 /app/node_modules 目錄。這樣可以確保容器內的 node_modules 目錄不會被本地的 node_modules 覆蓋。

這樣做的好處是，當你在本地開發時，本地的代碼會被映射到容器內，但依賴包（node_modules）仍然使用容器內的版本，避免了本地和容器之間的依賴包版本不一致問題。

##### 匿名卷的管理

匿名卷是自動創建和管理的，但你可以使用 Docker 命令來查看和刪除它們：

- 查看所有卷：

  ```bash
  docker volume ls
  ```

- 刪除一個卷：

  ```bash
  docker volume rm <volume_name>
  ```

匿名卷通常沒有特定的名稱，會顯示為一個自動生成的 ID。例如：

```bash
DRIVER              VOLUME NAME
local               1e2b3d4f5g6h7i8j9k0l
```

##### 總結

匿名卷是一種方便的方法來保護容器內的特定目錄，使其不會被本地目錄覆蓋，從而保持依賴包或配置文件的一致性和完整性。在 Docker 命令中使用匿名卷，可以幫助你在開發和部署應用程序時管理和保護數據。

#### Named Volume 具名卷

具名卷（Named Volume）是 Docker 中的一種卷，用於數據持久化和共享。具名卷有一個指定的名稱，這使得它們易於管理和引用。具名卷的主要目的是在容器之間共享數據，並且在容器刪除後數據依然保留。

##### 具名卷的特點

1. 持久化數據：數據保存在 Docker 管理的地方，容器刪除後數據仍然存在。
2. 易於管理：具名卷有一個名稱，可以通過名稱來管理和引用它們。
3. 共享數據：多個容器可以使用同一個具名卷來共享數據。

##### 使用具名卷的命令

###### 創建具名卷

你可以使用以下命令創建一個具名卷：

```bash
docker volume create my_named_volume
```

這個命令會創建一個名為 my_named_volume 的卷。

###### 使用具名卷

當你運行容器時，可以使用 -v 參數將具名卷掛載到容器內的某個目錄：

```bash
docker run -p 5137:5137 -v my_named_volume:/app/node_modules react-docker
```

這個命令將 my_named_volume 掛載到容器內的 /app/node_modules 目錄。

###### 管理具名卷

你可以使用以下命令來管理具名卷：

- 查看所有卷：

  ```bash
  docker volume ls
  ```

- 查看具名卷的詳細信息：

  ```bash
  docker volume inspect my_named_volume
  ```

- 刪除具名卷：

  ```bash
  docker volume rm my_named_volume
  ```

##### 示例

假設你有一個 React 應用，你希望在運行容器時使用具名卷來持久化和管理依賴包，防止本地 node_modules 覆蓋容器內的 node_modules。你可以這樣做：

1. 創建具名卷

   ```bash
   docker volume create react_node_modules
   ```

2. 使用具名卷運行容器

   ```bash
   docker run -p 5137:5137 -v "${pwd}:/app" -v react_node_modules:/app/node_modules react-docker
   ```

這樣，具名卷 react_node_modules 會被掛載到容器內的 /app/node_modules 目錄，確保容器內的 node_modules 目錄不會被本地的覆蓋，並且依賴包數據會被持久化和復用。

##### 總結

具名卷是一種用於數據持久化和共享的 Docker 卷。通過使用具名卷，你可以確保數據在容器的生命周期之外持續存在，並且可以在多個容器之間共享數據。這使得具名卷在構建可靠且可擴展的容器化應用程序中非常有用。

#### 匿名卷 VS 具名卷

匿名卷和具名卷各有其適用場景。匿名卷適合快速和臨時的使用，而具名卷更適合需要持久化數據和跨容器共享數據的場景。具名卷提供了更多的管理便利和靈活性，是大多數情況下更推薦的選擇。

#### 總結

我們的需求是修改主機檔案的同時更新容器內的檔案，需要用到 Bind Mount 將當前目錄掛載到容器內的 /app

但這時本地的 node_modules 目錄會覆蓋容器內的 /app/node_modules 目錄，而本地的依賴包版本可能與容器內需要的不一致，特別是在跨平台時，項我們本地是 MAC OS 但是容器內是 alpine linux 就會出錯，所以要使用匿名卷保護容器內的 node_modules 目錄

組合起來的指令就會是： `docker run -p 5173:5173 -v "$(pwd):/app" -v /app/node_modules react-docker`

解釋：用 docker 啟動一個容器，映射主機的 5173 port 到容器 的 5173 port ， mount 主機的 $(pwd) 到容器的 /app ，用匿名卷保護容器的 /app/node_modules 不被 mount 的時候覆蓋，容器用的是 react-docker 這個 image

### Push Image To DockerHub

注意：要推到 dockerhub 的 image name 格式一定要是 `DOCKERHUB_USERNAME/IMAGE_NAME[:TAG]` ， e.g. : `andygithub91/react-docker:1.0`

1. use command line :

   1. login: `docker login`

   2. attach tag to image `docker tag IMAGE_NAME DOCKERHUB_USERNAME/IMAGE_NAME` , e.g.: `docker tag react-docker andygithub91/react-docker`

      username 可以到 dockerhub 網站點擊右上角頭像，會看到你的 username

      docker tag 是 Docker 的一個命令，用於創建一個現有映像的新標籤。

      react-docker 是已經存在的 Docker 映像的名稱（或標籤）。

      andygithub91/react-docker 是你想要給這個映像添加的新標籤。

      這個命令不會創建新的映像，而是給現有的 react-docker 映像一個新的標籤 andygithub91/react-docker。

      一個 Docker 映像可以有很多標籤。

   3. push image to docker hub: `docker push IMAGE_NAME` , e.g.: `docker push andygithub91/react-docker`
   4. 到 Docker Desktop 點擊左側欄 Images -> 點擊 Hub 看到我們剛剛 push 的 image andygithub91/react-docker 就成功了，到 https://hub.docker.com/ 也會看到剛剛 push 的 image andygithub91/react-docker

2. use Docker Desktop
   1. 右上角點頭像登入
   2. `docker tag IMAGE_NAME DOCKERHUB_USERNAME/IMAGE_NAME`, e.g.: `docker tag react-docker andygithub91/react-docker`
   3. 到 Docker Desktop 點擊左側欄 Images -> 找到你要 push 的 image 的最右邊有個 ... 點擊他 -> 點擊 Push to Hub

## Docker Compose - React Demo

### Geting Started

`npm create vite@latest vite-project` -> select react -> select typescript : create a react project

`cd vite-project`

https://docs.docker.com/reference/cli/docker/init/
`docker init` Creates Docker-related starter files for your project

```sh
vite-project % docker init

Welcome to the Docker Init CLI!

This utility will walk you through creating the following files with sensible defaults for your project:
  - .dockerignore
  - Dockerfile
  - compose.yaml
  - README.Docker.md

Let's get started!

? What application platform does your project use? Node
? What version of Node do you want to use? 20.10.0
? Which package manager do you want to use? npm
? Do you want to run "npm run build" before starting your server? No
? What command do you want to use to start the app? npm run dev
? What port does your server listen on? 5173

CREATED: .dockerignore
CREATED: Dockerfile
CREATED: compose.yaml
CREATED: README.Docker.md

✔ Your Docker files are ready!
```

Dockerfile 用我們在 react-docker/Dockerfile 寫好的就可以了，把 react-docker/Dockerfile 的內容複製起來貼到 vite-project/Dockerfile

改寫 vite-project/compose.yaml

```yaml
services: # 這是 Docker Compose 文件的主要部分，用於定義服務（容器）。
  web: # 服務名稱
    build: # 建構相關設定
      context: . # 使用當前目錄作為建構上下文。上下文（context）在 Docker 中是指 Docker 引擎在建構映像時可以訪問的所有文件和目錄。在 Docker Compose 文件中，上下文指定了 Docker 應該在哪個目錄中查找 Dockerfile 和其他相關文件。
    ports:
      - 5173:5173 # 將宿主機的 5173 端口映射到容器內的 5173 端口
    volumes:
      - .:/app # 將宿主機當前目錄掛載到容器內的 /app 目錄
      - /app/node_modules # 確保容器內的 /app/node_modules 目錄保持獨立，避免宿主機的 node_modules 干擾
```

run `docker compose up` 連到 http://localhost:5173/ 出現畫面就代表成功了

## Docker Compose - MERN Demo

### Getting Started

copy `starter_mern-docker` directory to `/` and rename it to `mern-docker`

copy `react-docker/Dockerfile` directory to `mern-docker/frontend/Dockerfile` ，把原本的註解移除，註解掉操作用戶和群組權限的命令，讓他簡單一點，因為我們想專注在 docker compose 或其他地方

add `node_modules` to `mern-docker/frontend/.dockerignore`

add `mern-docker/backend/Dockerfile`

```Dockerfile
FROM node:20-alpine
# RUN addgroup app && adduser -S -G app app
# USER app
WORKDIR /app
COPY package*.json ./
# USER root
# RUN chown -R app:app .
# USER app
RUN npm install
COPY . .
EXPOSE 8000
CMD npm start
```

add `node_modules` to `mern-docker/backend/.dockerignore`

add `mern-docker/compose.yaml` and see the code in `mern-docker/compose.yaml`

run `docker compose up` 會啟動三個容器，連上 http://localhost:5173/ 看到畫面就成功了，但是到 mern-docker/frontend/src/App.jsx 更新程式後發現畫面沒有即時更新

開一個新的終端機不要把 docker compose up 這個終端機關了，剛剛畫面沒有即時更新是因為我們沒有用 watch 命令，要監視檔案變動的話應該 run `docker compose watch` ，這時更新程式後會發現畫面有即時更新了

來測試看看 backend 是不是也有被監視，開一個新的終端機 `cd mern-docker/backend` 安裝改變終端機字體顏色的 package ， run `npm i colors`，會看到原本運行 docker compose watch 的終端機在跑一些東西，這是因為我們安裝了新的 package 被 docker 監視到了，所以幫我們 rebuild api service

我們開啟 mern-docker/backend/index.js 會看到

```js
const colors = require("colors");
app.get("/", (req, res) => {
  console.log("Hello World!".rainbow);

  res.send("Hello World!");
});
```

所以我們訪問 http://localhost:8000/ 再回去看 compose up 的終端機會發現終端機輸出 api-1 | Hello World! 而且 Hello World! 是彩虹色的，因為我們有用 compose watch 指令監視 mern-docker/backend/package.json ，我們剛才 run `npm i colors` 的時候 mern-docker/backend/package.json 有被更動，觸發 compose watch 去 rebuild api service ，所以 mern-docker/backend/index.js 中寫到關於 colors package 的代碼才會生效

#### volumes in Docker Compose

Docker 中有多種不同類型的卷（volumes），每種卷都有其特定的用途和特性。主要有以下幾種：

1.  Host Volumes（主機卷）：

    這種卷直接將主機文件系統上的一個特定目錄或文件掛載到容器內部。這在開發和測試環境中特別有用，因為它允許您在容器外部更改文件，這些更改會立即反映在容器內部。
    配置示例：

    ```yaml
    Copy code
    services:
      web:
        image: nginx:latest
        volumes:
          - ./host_path:/container_path
    ```

2.  Named Volumes（命名卷）：

    這是由 Docker 管理的卷，您只需給它們指定一個名稱。這種卷對於數據持久性非常有用，因為 Docker 會自動管理卷的存儲位置。
    配置示例：

    ```yaml
    services:
      web:
        image: nginx:latest
        volumes:
          - webdata:/usr/share/nginx/html
    volumes:
      webdata:
    ```

3.  Anonymous Volumes（匿名卷）：

    匿名卷（Anonymous Volumes）是 Docker 中的一種卷類型，這種卷不會被顯式命名，因此會由 Docker 自動生成一個唯一的名稱。這種卷常用於臨時數據存儲，或在短期運行的容器中使用。

    特點
    自動生成名稱：Docker 會自動生成一個唯一的名稱來識別這些卷。

    簡單配置：在 Docker Compose 中配置匿名卷非常簡單，只需指定容器內的目標路徑即可。
    臨時性：這種卷適合存儲臨時數據，不需要持久化或與其他容器共享的數據。
    配置示例
    以下是如何在 Docker Compose 中配置匿名卷的示例：

    ```yaml
    version: "3"
    services:
      web:
        image: nginx:latest
        volumes:
          - /usr/share/nginx/html
    ```

    在這個例子中，/usr/share/nginx/html 目錄被配置為匿名卷。這意味著 Docker 會自動生成一個名稱來掛載這個卷，並將它連接到容器內部的 /usr/share/nginx/html 路徑。

    使用場景
    臨時數據存儲：適合用於不需要持久化的臨時數據存儲。
    測試和開發：在開發和測試環境中，可以快速配置和使用匿名卷，而不需要關心卷的名稱和管理。
    如何查看匿名卷
    匿名卷可以通過以下命令查看：

    ```sh
    docker volume ls
    ```

    這將列出所有的 Docker 卷，包括匿名卷。匿名卷的名稱通常是一個由 Docker 自動生成的隨機字符串。

    匿名卷的清理
    由於匿名卷沒有顯式的名稱，當容器停止或刪除後，這些卷可能會留在系統中，佔用磁盤空間。可以使用以下命令清理未被使用的匿名卷：

    ```sh
    docker volume prune
    ```

    這將刪除所有未被容器使用的卷，包括匿名卷。

    總結
    匿名卷是 Docker 中一種方便的卷類型，用於存儲臨時數據。它們由 Docker 自動管理和生成名稱，適合在開發、測試和臨時數據存儲的場景中使用。需要注意的是，應定期清理不再使用的匿名卷，以避免佔用過多的磁盤空間。

4.  Bind Mounts（綁定掛載）：

    綁定掛載類似於主機卷，允許您將主機上的一個特定文件或目錄綁定到容器內部。不同之處在於，綁定掛載允許您指定更細粒度的設置，例如只讀權限。
    配置示例：

    ```yaml
    services:
      web:
        image: nginx:latest
        volumes:
          - type: bind
            source: ./host_path
            target: /container_path
            read_only: true
    ```

5.  tmpfs Mounts（臨時文件系統掛載）：

    這種掛載將數據存儲在主機的內存中，而不是持久存儲在磁盤上。這在需要高性能且不需要持久存儲的情況下非常有用。
    配置示例：

    ```yaml
    services:
      web:
        image: nginx:latest
        volumes:
          - type: tmpfs
            target: /container_path
    ```

每種卷都有其特定的使用情境和優缺點。根據您的應用需求，可以選擇合適的卷類型來確保數據的持久性、性能和管理便利性。

#### Docker Scout

Docker Scout 是一個類似弱掃的工具，拿來掃你 image 的弱點

到 Docker Desktop -> 左側欄點擊 Docker Scout -> 點擊 Sample image 下拉選單選擇 mern-docker-web:latest -> 點擊 View packages and CVEs 就可以看到對於我們 image 的弱掃報告

## Full Stack Next.js 14 Demo

### Getting Started

copy /starter_next-docker paste to root directory and rename to /next-docker

run cd `cd next-docker` and run `docker init`

```sh
? What application platform does your project use? Node
? What version of Node do you want to use? 20.10.0
? Which package manager do you want to use? npm
? Do you want to run "npm run build" before starting your server? No
? What command do you want to use to start the app? npm run dev
? What port does your server listen on? 3000

CREATED: .dockerignore
CREATED: Dockerfile
CREATED: compose.yaml
CREATED: README.Docker.md
```

修改 next-docker/Dockerfile ，詳細的代碼請看 next-docker/Dockerfile

修改 next-docker/compose.yaml ，詳細的代碼請看 next-docker/compose.yaml

### 怎麼連到 MongoDB service(container)

MongoDB 的 connect string 的格式： `mongodb://username:password@hostname:port/database?authSource=admin`

`mongodb://` 是協議識別符，表明這是一個 MongoDB 連接 URL

`username:password@` username: 用於認證的用戶名。password: 該用戶的密碼。@ 符號分隔認證信息和主機信息。

`hostname` 是 MongoDB 服務器的主機名或 IP 地址。在 Docker Compose 創建的網絡中， service 名稱可以直接用作主機名。

`:port` 是 MongoDB 服務器的端口號。27017 是 MongoDB 的默認端口。

`/database` 指定要連接的數據庫名稱。

`?` 標誌著 URL 參數的開始。

`authSource=admin` 指定存儲用戶憑證的數據庫。admin 是常用的認證源數據庫。

所以在我們前端的環境變數就會這樣寫

```yaml
    environment:
      - DB_URL=mongodb://andy:test@db:27017/myapp?authSource=admin

  db:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=andy
      - MONGO_INITDB_ROOT_PASSWORD=test
```

因為 next-docker/.env.example 裡面有寫到 `DB_URL=` ，所以這個環境變數後面接的就是 MongoDB 的 connect string

`mongodb://username:password@hostname:port/database?authSource=admin` 帶入參數後 -> `mongodb://andy:test@db:27017/myapp?authSource=admin` ，`mongodb://` 是協議識別符，`andy` 是用戶名，`test` 是密碼，`@` 是用戶名，`@` 分隔認證信息和主機信息的符號，`db` 是 service ，`:27017` 是 MongoDB 服務器的端口號，`/myapp` 是數據庫名稱，`?` 標誌著 URL 參數的開始，`authSource=admin` 指定存儲用戶憑證的數據庫。admin 是常用的認證源數據庫。

### 如何刪除 volume
stop container -> rm container -> rm volume