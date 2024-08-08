# https://hub.docker.com/layers/library/node/20-alpine/images/sha256-aa22e2c263d7242e7452d1efa40a031ac8ccf2305cc4d925c737486d339bd178?context=explore
# alpine is a lightweight version of linux
FROM node:20-alpine

# WORKDIR 指定我們要 run commands 的 dir
# /app 是 standard convention
WORKDIR /app

# COPY source dest <- 語法
# `COPY . .` will copy everything from our current directory to the docker image ，第一個點表示本地機器的當前目錄的路徑，第二個點表示 container 的當前目錄的路徑
COPY . .

# CMD 指定我們要執行的命令
CMD node hello.js