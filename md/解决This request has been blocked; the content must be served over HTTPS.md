# <center>解决This request has been blocked; the content must be served over HTTPS

> 在 HTTPS 承载的页面上不允许出现 http 请求，一旦出现就是提示或报错：
This request has been blocked; the content must be served over HTTPS  

## 方法1：  
在 html 文件头中添加

    ```css
    <meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests"/>
    ```

## 方法2：  
在 nginx 的 server 模块添加

    ```bash
    add_header Content-Security-Policy "upgrade-insecure-requests;connect-src *";
    ```
