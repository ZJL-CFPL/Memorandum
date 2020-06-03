# Linux下CentOs,配合Filezilla构建

## CentOs下安装Node.js
 +  wget命令下载Node.js安装包
    - 下载地址： [https://nodejs.org/dist/]
    - 安装命令： wget https://nodejs.org/dist/v8.16.2/node-v8.16.2-linux-x64.tar.xz
    - 解压文件： tar xvf node-v8.16.2-linux-x64.tar.xz
    - 创建软链接，使node和npm命令全局有效： 
      + ln -s /root/node-v8.16.2-linux-x64/bin/node /usr/local/bin/node
      + ln -s /root/node-v8.16.2-linux-x64/bin/npm /usr/local/bin/npm
    - 查看安装后版本
      + node -v
      + npm -v
  + 安装GIT: yum install git
  + 安装pm2(node端进程管理工具)
    - npm install -g pm2
    - 创建软连接，便于在任意目录下都可以直接使用pm2命令： 
      + ln -s /root/node-v8.16.2-linux-x64/bin/pm2 /usr/local/bin/pm2
    - pm2 使用：
      + cd 进入到项目目录
      + pm2 start npm --name "nuxtSSR" -- run start 
        - 这里的name对应的是package.json中的项目名称
  + 某些npm安装包可使用 淘宝镜像 cnpm加快安装速度
    - npm install -g cnpm --registry=https://registry.npm.taobao.org
    - 添加全局软连接
      + ln -s /usr/local/nodejs/bin/cnpm /usr/local/bin/cnpm

--------------------------------------------------------------

## Nuxt 服务端配置
  + cd 进入到项目文件夹
  + 增加nuxt项目
  + nuxt项目安装
    - npm install
    - npm run build
    - npm run start
    - pm2进行进程管理： pm2 start ./node_modules/nuxt/bin/nuxt
      + 测试： curl http://localhost:3000   // 出来一段应用html说明部署成功
  + 使用nginx代理，发布外网
    - ``` 
        upstream nuxt {
          server 127.0.0.1:3000;
        }
        server{
          listen 80;
          server_name 47.112.180.235;
          location / {
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;  
              proxy_set_header Connection "upgrade";
              proxy_set_header Host $host;
              proxy_set_header X-Nginx-Proxy true;
              proxy_cache_bypass $http_upgrade;
              proxy_pass http://nuxt; #反向代理
          }
        }
      ```
    + 外网访问应用:  http://47.112.180.235/80

-----------------------------------------------------------------

### pm2相关命令
  + pm2 start nuxt  //启动server.js进程
  + pm2 start nuxt -i 4 //启动4个server.js进程
  + pm2 restart nuxt //重启server.js进程
  + pm2 stop all //停止所有进程
  + pm2 stop nuxt //停止server.js进程
  + pm2 stop 0  //停止编号为0的进程
  + pm2 list //查看
  + pm2 monit //监控
  + pm2 logs //查看日志

### nginx相关(通过systemctl管理)
  + 安装：
    1. sudo yum install nginx
    2. sudo systemctl enable nginx 设置开启启动(会自动创建软连接关联nginx)
    3. sudo systemctl start nginx 启动nginx
    4. sudo systemctl status nginx 查看运行状态
  + 配置路径：
    - /etc/nginx/ 
    - /etc/nginx/nginx.conf
  + 命令： 
    - sudo systemctl start nginx // 启动 Nginx
    - sudo systemctl stop nginx // 停止 Nginx
    - sudo systemctl restart nginx // 重启 Nginx
    - sudo systemctl reload nginx // 修改 Nginx 配置后，重新加载
    - sudo systemctl enable nginx // 设置开机启动 Nginx
    - sudo systemctl disable nginx // 关闭开机启动 Nginx
    - sudo systemctl status nginx // 查看运行状态

