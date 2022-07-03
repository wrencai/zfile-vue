# Z-File

基于 [https://github.com/zhaojun1998/zfile-vue](https://github.com/zhaojun1998/zfile-vue) 网盘改造.
zfile原项目预览地址: [http://zfile.vip](http://zfile.vip)

此项目是一个在线文件目录的程序, 支持本地存储, 使用定位是为个人简易NAS服务. 

后端代码: [https://github.com/wrencai/zfile](https://github.com/wrencai/zfile)

## 访问地址

用户前台: http://127.0.0.1:9080/main

初始安装: http://127.0.0.1:9080/install

管理后台: http://127.0.0.1:9080/admin

# 启动方式
## 第一次启动时安装依赖
npm install
## 本地启动调试命令，需要配合启动后台项目，可以在vue.config.js中的proxy项来配置后台地址进行接口代理
npm run serve
## 构建命令，构建完成后，将dist文件夹内容拷贝到后端项目src/main/resources/static下面替换即可进行前端文件更新
npm run build
