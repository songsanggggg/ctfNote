# 工具合集

## 字典

- **Fuzzdb**https://github.com/fuzzdb-project/fuzzdb

  里面除了一些字典还包含简单的🐎
  
- **Rockyou**很大的字典

## 泄漏

### Git泄漏

- **GitHack**https://github.com/lijiejie/GitHack

  不是很好用，说有坑

- **GitHackerhttps://github.com/WangYihang/GitHacker**

### Svn泄漏

- **svnExploit**https://github.com/admintony/svnExploit

### DS_Store泄漏

- **ds_store_exp**https://github.com/lijiejie/ds_store_exp

### CVS/SVN/Bazaar/bzr/Mercurial/HG/GIT

- **dvcs-ripper**https://github.com/kost/dvcs-ripper

## 爆破

### Zip爆破

- **ZipCracker**https://github.com/asaotomo/ZipCracker

### JWT爆破

- **Hashcat**直接从arch源安装`hashcat -a 0 -m 16500 <jwt> <wordlist>`

- **c-jwt-cracker**https://github.com/brendan-rius/c-jwt-cracker.git

  ```shell
  docker run -it --rm  jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.cAOIAifu3fykvhkHpbuhbvtH807-Z2rI1FS3vX1XMjE
  ```

  

## Flask session

- **Flask-unsign**https://github.com/Paradoxis/Flask-Unsign

  该工具还支持爆破

- **flask-session-cookie-manager**https://github.com/noraj/flask-session-cookie-manager

## SSTI

- **fenjing**https://github.com/Marven11/Fenjing
- **SSTImap**https://github.com/vladko312/SSTImap

## Hash长度拓展

- **Hash-ext-attack**https://github.com/shellfeel/hash-ext-attack

## 目录

- **dirb**https://github.com/v0re/dirb
- **dirsearch**https://github.com/maurosoria/dirsearch

## 绕过

### bash命令绕过

- **bashFuck**https://github.com/ProbiusOfficial/bashFuck

## burp插件

TsojanScan	https://github.com/Tsojan/TsojanScan	一个集成的BurpSuite漏洞探测插件
ShiroScan	https://github.com/sv3nbeast/ShiroScan	Shiro<=1.2.4反序列化，一键检测工具
FastjsonScan	https://github.com/a1phaboy/FastjsonScan	Fastjson扫描器，可识别版本、依赖库、autoType状态等
knife	https://github.com/bit4woo/knife	添加一些右键菜单让burp用起来更顺畅
HaE	https://github.com/gh0stkey/HaE	HaE 请求高亮标记与信息提取的辅助型 BurpSuite 插件
captcha-killer-modified	https://github.com/f0ng/captcha-killer-modified	验证码识别
BurpCrypto	https://github.com/whwlsfb/BurpCrypto	支持多种加密算法或直接执行JS代码的用于爆破前端加密的BurpSuite插件
autoDecoder	https://github.com/f0ng/autoDecoder	Burp插件，根据自定义来达到对数据包的处理（适用于加解密、爆破等）
AutoRepeater	https://github.com/nccgroup/AutoRepeater	自动发送请求
jsEncrypter	https://github.com/c0ny1/jsEncrypter	一个用于前端加密Fuzz的Burp Suite插件
APIKit	https://github.com/API-Security/APIKit	可以主动/被动扫描发现应用泄露的API文档
RouteVulScan	https://github.com/F6JO/RouteVulScan	递归式被动检测脆弱路径的burp插件
json-web-tokens	https://github.com/portswigger/json-web-tokens（burp商店可下载）	JWT测试
Log4j2Scan	https://github.com/whwlsfb/Log4j2Scan	被动扫描Log4j2漏洞CVE-2021-44228的BurpSuite插件
burp-awesome-tls	https://github.com/sleeyax/burp-awesome-tls	绕过WAF，欺骗任何浏览器。
ViewStateDecoder	https://github.com/raise-isayan/ViewStateDecoder	Burpsuite 扩展。支持 ASP.NET ViewStateDecoder
chunked-coding-converter	https://github.com/c0ny1/chunked-coding-converter	添加脏数据和延时分块传输
xia_sql	https://github.com/smxiazi/xia_sql	xia SQL (瞎注) burp 插件 ，在每个参数后面填加一个单引号，两个单引号，一个简单的判断注入小插件。

