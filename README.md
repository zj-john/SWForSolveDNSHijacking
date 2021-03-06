# SWForSolveDNSHijacking
使用service worker缓解DNS被劫持的问题

## 功能
通过service worker的配置，在域名被劫持或其它无响应的情况下，通过config文件中配置的备用域名进行替换访问，备用域名访问成功后会把资源加入缓存。

## 目录结构
```
│  .gitignore
│  dnsconfig.json
│  README.md
│  service-worker.js
│
├─demo
  │  app.js
  │  package.json
  │
  ├─certificate
  │      ca.cer
  │      csr.pem
  │      private.pem
  │
  └─public
      │  dnsconfig.json
      │  index.html
      │  service-worker.js
      │
      ├─css
      │      style.css
      │
      └─js
              main.js
```
整个目录结构分为2部分，

### 主要功能部分

#### service-worker.js

在你的项目根文件(假设为index.html)中加入如下代码：
```html
<script>
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('service-worker.js').then(function(reg) {
      if(reg.installing) {
        console.log('Service worker installing');
      } else if(reg.waiting) {
        console.log('Service worker installed');
      } else if(reg.active) {
        console.log('Service worker active');
      }
    }).catch(function(error) {
      console.log('Registration failed with ' + error);
    });
  }
</script>
```
更精简的版本：
```html
<script>
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('service-worker.js');
  }
</script>
```

把service-worker.js 拷贝到项目根目录下即可。

#### dnsconfig.json
域名主备配置
```
{
  "webresource.c-ctrip.com":{
    "backupType": "domain",
    "backup":["webresource.ctrip.com"],
    "addCache":true
  }
}
```
使用时，把config配置为自己需要的内容，拷贝到项目根目录下即可。

### demo
demo文件夹下是一个可以运行的，基于node，使用https的web站点。

[地址](https://zj-john.github.io/RelieveDNSHijackingbySW/demo/public/index.html)

使用方式：  
通过修改hosts等方式，使目前访问的webresource.c-ctrip.com 无法访问。站点将从webresource.ctrip.com获取资源，并缓存。

#### chrome环境配置：  
本地运行的https是不安全的，所以要想service-worker能够成功运行，需要配置下浏览器的启动参数。

windows下启动参数配置如下:
```
C：\ Program Files（x86）\ Google \ Chrome \ Application \ chrome.exe --ignore-certificate-errors --unsafely-treat-insecure-origin-as-secure= https://localhost:8001
```

MacOS：
```
/ Applications / Google \ Chrome.app/Contents/MacOS/Google \ Chrome --user-data-dir = / tmp / foo --ignore-certificate-errors --unsafely-treat-insecure-origin-as-secure = https://localhost:8001
```

> 关于此配置的更多内容请参考：https://deanhume.com/testing-service-workers-locally-with-self-signed-certificates/
