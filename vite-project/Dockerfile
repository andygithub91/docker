# set the base image to create the image for react app
FROM node:20-alpine

# create a user with permissions to run the app
# -S -> create a system user
# -G -> add the user to a group
# This is done to avoid running the app as root
# If the app is run as root, any vulnerability in the app can be exploited to gain access to the host system
# It's a good practice to run the app as a non-root user


# 為了避免有些 user 不好的操作行為，需要設定一些權限來保護容器。

# 命令解釋：
# addgroup app:
# addgroup 是用來創建一個新的群組。
# app 是新群組的名稱。
# 所以這個命令會創建一個名為 app 的群組。

# adduser -S -G app app:
# adduser 是用來創建一個新的用戶。
# -S 參數表示創建一個系統用戶，這個用戶通常是沒有登錄權限的，並且用來運行系統服務或守護進程。
# -G app 參數指定新用戶將會被添加到名為 app 的群組中。
# 最後的 app 是新用戶的名稱。
# 所以這個命令會創建一個名為 app 的系統用戶，並且將該用戶添加到名為 app 的群組中。

# 這兩個命令是用來創建一個新的群組和一個新的系統用戶，並將該用戶加入該群組。這在設置服務或應用程式時很常見，可以用來管理權限和隔離不同的服務。
# 這可以避免用 root user 去 run app ，如果我們用 root 身份 run app 的話，可能有人會利用 app 的漏洞 access 進 host system 。
# 創建新的群組和用戶，而不是直接使用 root，有重要的安全方面的原因：
# 最小權限原則：這是一個重要的安全原則，意味著應該只給予程序或用戶執行其任務所需的最小權限。使用 root 用戶運行程序意味著該程序擁有整個系統的最高權限，如果該程序被攻擊者利用，整個系統都可能被攻擊。
# 減少風險：如果某個應用程序運行在一個特定的用戶下，並且該應用程序出現漏洞，攻擊者只能取得該用戶的權限，而不能取得整個系統的控制權。
RUN addgroup app && adduser -S -G app app


# RUN su app 和 USER app 一樣嗎？
# USER app 指令設置了接下來所有 Dockerfile 指令（包括 CMD 和 ENTRYPOINT）將以指定的用戶身份運行。這是一個全局設置，並且會持續到 Dockerfile 結束。
# RUN su app 指令是用來在單個 RUN 指令內切換用戶身份的。這通常用於臨時切換用戶身份來執行某些命令。需要注意的是，這種方法並不會影響 Dockerfile 中後續的指令，它只在當前的 RUN 指令中有效。

# set the user to run the app
USER app

# WORKDIR /app 和 RUN cd /app 是不一樣的
# WORKDIR /app 設置了一個持久的工作目錄。當你設置了 WORKDIR，後續的所有指令（RUN、CMD、ENTRYPOINT、COPY、ADD）都將在這個目錄中執行，直到再次改變工作目錄。
# RUN cd /app 僅在該 RUN 指令執行期間改變工作目錄。一旦該指令執行完畢，工作目錄會重置回之前的狀態。
# set the working directory to /app
WORKDIR /app

# copy package.json and package-lock.json to the working directory
# This is done before copying the rest of the files to take advantage of Docker’s cache
# If the package.json and package-lock.json files haven’t changed, Docker will use the cached dependencies
COPY package*.json ./

# sometimes the ownership of the files in the working directory is changed to root
# and thus the app can't access the files and throws an error -> EACCES: permission denied
# to avoid this, change the ownership of the files to the root user
USER root

# change the ownership of the /app directory to the app user
# chown -R <user>:<group> <directory>
# chown command changes the user and/or group ownership of for given file.
RUN chown -R app:app .

# change the user back to the app user
USER app

# install dependencies
RUN npm install

# copy the rest of the files to the working directory
COPY . .

# expose port 5173 to tell Docker that the container listens on the specified network ports at runtime
EXPOSE 5173

# command to run the app
CMD npm run dev