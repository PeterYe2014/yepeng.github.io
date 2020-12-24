README.md

https://imququ.com/post/readme.html

本站主要用到的技术及主要支持的特性，罗列如下：

·         **Blog System**

o    ThinkJS，2.x.x，本站使用的 Node.js 服务端全能框架，[详情](https://imququ.com/post/ququblog.html)；

o    Node.js，4.x.x LTS，ThinkJS 运行环境；

o    MySQL，5.x.x，数据库；

o    Memcached，缓存热门数据；

o    ElasticSearch，提供博客站内搜索，[详情](https://imququ.com/post/elasticsearch.html)；

·         **Deployment**

o    Ubuntu，16.04.1 LTS；

o    Docker，本站使用 Docker Compose 提供运行环境，[详情](https://imququ.com/post/use-docker.html)；

·         **Web Server**

o    Nginx，1.11.x，[详情](https://imququ.com/post/my-nginx-conf.html)；

o    nginx-ct，1.3.x，提供 Certificate Transparency 功能，支持多证书配置；

o    ngx_brotli，提供对 [Brotli](https://github.com/google/brotli) 压缩格式的支持，具有更高压缩比；

o    OpenSSL，1.0.2，只启用 TLSv1~v1.2，不支持 Windows XP IE6~7；

o    VeryNginx，1.3.3，提供强大的流量干预功能，[详情](https://imququ.com/post/use-verynginx.html)；

·         **HTTPS & HTTP/2**

o    Let's Encrypt，免费好用的证书，[详情](https://imququ.com/post/letsencrypt-certificate.html)；

o    RSA/ECC Certificate，优先使用 ECC 证书，体积更小，[详情](https://imququ.com/post/ecc-certificate.html)；

o    Certificate Transparency，证书透明度，[详情](https://imququ.com/post/certificate-transparency.html)；

o    HTTP Strict Transport Security，HSTS，[详情](https://imququ.com/post/sth-about-switch-to-https.html#toc-2)；

o    HSTS Preloading，加入浏览器内置 HSTS 列表；

o    Public Key Pinning，HPKP，[详情](https://imququ.com/post/http-public-key-pinning.html)；

o    OCSP Stapling；

o    Session Resumption，Identifier/Tickets；

·         **Security**

o    CSP2，Content Security Policy Level 2，[详情](https://imququ.com/post/content-security-policy-level-2.html)；

o    2FA，Two-factor Authentication，用于后台登录，[详情](https://imququ.com/post/about-two-factor-authentication.html)；

o    Security Response Header，使用 HTTP 响应头部增强网站安全，[详情](https://imququ.com/post/web-security-and-response-header.html)；

·         **Experience**

o    Mobile friendly；

o    Google PWA && AMP，[示例](https://imququ.com/amp/post/readme.html)；

o    Markdown，本站所有文章都支持以 Markdown 格式查看，[示例](https://imququ.com/post/readme.md)；

o    Kindle eBook，你可以使用 Kindle 阅读本站，[详情](https://imququ.com/post/for-kindle.html)；

o    Full Text RSS，[欢迎订阅](https://imququ.com/rss.html)；

o    PubSubHubbub，让订阅网站实时感知新文章；

o    WebP，优先使用 WebP 格式图片，减小图片体积；

·         **Third-Party Service**

o    UptimeRobot，监测本站服务是否可用，[详情](https://stats.uptimerobot.com/nrKx9T6NZ)；

o    LetsMonitor.org，监测本站所用证书是否临近过期；

o    Disqus，为了让国内用户能流畅使用，进行了特殊处理；

o    Google Analytics，为了国内正常使用，在服务端做了中转；

 