## 联调

主要介绍前后端人员在自己本地功能开发完成后如何进行联调。

#### 服务端人员

1. 设置联调域名，修改项目代码下的`bulid/nginx/vhosts/app_name.conf`文件，将`server_name`改为联调使用的域名。

   > 一般联调环境的域名以.d进行标识

2. 开启服务

   ```shell
   docker-compose up -d
   ```


#### 前端人员

1. 修改本地host，添加服务端人员机器的IP地址与域名映射。

   > 域名可以随意设置

2. 修改配置文件`config/webpack.compose.js`，添加联调的api代理，域名为刚刚设置的域名地址。

   ```javascript
   const PROXY = {
     // 联调
     '/demo': {
       target:'http://demo.server.com',
       changeOrigin: true,
     },
   }
   ```

3. 配置内网HTTP代理，设置自己本机的IP、端口和想要分配的域名（例如：http://demo.client.com）。

   参考[内网HTTP代理](/base/nginx.md)

4. 访问设置的代理域名即可访问，例如http://demo.client.com。