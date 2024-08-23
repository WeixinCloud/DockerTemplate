# 微信云托管场景化案例集合

在将项目迁移到微信云托管的过程中，因为缺少dockerfile等必要的构建文件，同时自己又没有相关的经验，导致迁移出现卡点。

此项目整理了一些常见的项目案例场景，可以根据自己的需要下载对应文件夹的内容，按照说明替换成自己的项目文件。

项目场景持续整理中，如果你有需要，可以直接填写[征集问卷](https://wj.qq.com/s2/10034327/8f93/)，向我们提交一些想法；如果是比较通用的，我们会收集进来。

## 一、项目清单

#### 1. [Nginx+HTML分发模板](./nginxdist/)

如果你有vue或者其他打包的web的项目，需要使用nginx分发，可以参考此模板，将其中的dist文件夹内容替换成自己的项目内容。

Nginx分发配置，可以直接修改根目录的 `my.conf`

#### 2. [Nginx+Node分发模板](./nginxnode/)

如果有一个node项目，同时需要nginx做反向代理，实现一些扩展需求，则可以参考此项目。

此处Dockerfile中最后CMD命令，是执行sh文件，可以在app/start.sh编写修改。

Nginx分发配置，可以直接修改根目录的 `my.conf`

#### 3. [PHP中SG11扩展安装模板](./phpsg11)

发问的人比较多，在这里直接写一下，给了一个php单一文件，如果是框架项目，需按需自己修改Dockerfile的安装步骤。

SG11的安装命令如下，可以在自己dockerfile的FROM下面一行添加下述命令，也可以整合你自己的安装步骤。

``` dockerfile
RUN PHP_VERSION=$(php -v | head -n1 | cut -d' ' -f2 | cut -d. -f1-2) \
    && mkdir -p /tmp/sourceguardian \
    && cd /tmp/sourceguardian \
    && curl -Os https://www.sourceguardian.com/loaders/download/loaders.linux-x86_64.tar.gz \
    && tar xzf loaders.linux-x86_64.tar.gz \
    && cp ixed.${PHP_VERSION}.lin "$(php -i | grep '^extension_dir =' | cut -d' ' -f3)/sourceguardian.so" \
    && echo "extension=sourceguardian.so" > /usr/local/etc/php/conf.d/15-sourceguardian.ini \
    && rm -rf /tmp/sourceguardian
```

#### 4. [jetty应用模板](./jetty)

如果你需要使用jetty来部署web应用，可以参考此案例。有些同学在podman上可以正常运行，迁移到云托管就一直启动不起来。

在这里需要注意，**微信云托管的k8s底层，使用的是docker，需要在我们提供的镜像在启动时运行一个守护进程(docker daemon)，而podman不需要；** 所以这里就产生了差异。

在这里我们提供了一个可以适用于云托管的docker模板，可以自行取用。

#### 5. [Nginx+java-openjdk分发模板](./nginxjava/)

如果有一个java项目，一般是springboot或者其他框架项目，构建产物为jar包，同时需要nginx做反向代理，实现一些扩展需求，则可以参考此项目。

此处Dockerfile中最后CMD命令，是执行sh文件，可以在 `dist/start.sh` 编写修改。

Nginx分发配置，可以直接修改 `dist/my.conf`

很多情况下，你需要将自己的域名根路径自动跳转成www开头，这种就可以使用nginx域名规则配置

比如，你想使用户在访问 `cloudbase.net` 时，自动路由为 `www.cloudbase.net`

nginx配置如下：

``` conf
# 前面的配置省略，请参考nginx规则
server {
    listen       80;
    server_name  cloudbase.net www.cloudbase.net;
    if ($host != 'www.cloudbase.net') {
        rewrite ^/(.*)$ http://www.cloudbase.net/$1 permanent;
    }
    location / {
        # 你的其他路由规则，按照你的需要配置
        # proxy_pass http://127.0.0.1:8080/;
    }
}
```

同时需要在云托管服务自定义域名，配置 `cloudbase.net` 和 `www.cloudbase.net` 指向此服务

#### 6. [Nginx+Node简易WebSocket](./websocket/)

使用Node.js构建了一个简单的WebSocket服务，直接链接最终域名根目录即可使用，用于测试微信云托管与客户端的连通性。

nginx转发配置，可以直接修改my.conf内容，启动命令在app/start.sh中修改

由于历史原因，部分服务提供的默认域名仍然不支持websocket，可以按照以下程式转换一下。

``` js
function getwsurl(envid,url){
    return url.replace(/[-][0-9]*[-]/,`-${envid}-`).replace('tcloudbase.com','wxcloudrun.com').replace('https://','wss://')
}

getwsurl('你的环境ID','服务默认域名')

// getwsurl('wxrun-demo','https://wss-923625-1300000000.ap-shanghai.run.tcloudbase.com')
// wss://wss-wxrun-demo-1300000000.ap-shanghai.run.wxcloudrun.com
```

## 二、贡献者征集

欢迎有经验的开发者来贡献自己认为比较常用的项目包，直接提交PR，我们会在review后合并到项目里。

- [zirali李冠宇](https://github.com/Tcloudbase)
