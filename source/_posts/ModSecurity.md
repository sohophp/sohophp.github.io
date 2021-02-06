---
title: ModSecurity
date: 2021-02-02 11:52:06
tags: [ModSecurity,CRS,Linux,CentOS,Apache,Httpd]
categories: [Linux,CentOS]

---

##  安装 ModSecurity

https://www.modsecurity.org/

```Bash
# 安装mod_security
$ yum install mod_security
# 
# 重启httpd
$ sudo systemctl restart httpd
```
##  安装 CRS (CoreRuleSet) 规则集

https://coreruleset.org/installation/  


<!--more-->
```Bash

# 以下以root运行
$ su
# 安装 wget
$ sudo yum install wget 
# 进入/etc/httpd下
$ cd /etc/httpd
# 下载CRS最新版
$ wget https://github.com/coreruleset/coreruleset/archive/v3.3.0.tar.gz
# 解压缩
$ tar -zxvf v3.3.0.tar.gz
# 创建动态链接
$ ln -s coreruleset-3.3.0 /etc/httpd/crs
# 复制配置文件
$ cp crs/crs-setup.conf.example crs/crs-setup.conf
# 修改conf
$ sudo vim /etc/httpd/conf.d/mod_security.conf
# 加两行
> IncludeOptional crs/crs-setup.conf
> IncludeOptional crs/rules/*.conf
# 检查 httpd配置是否有错
$ apachectl configtest
# 重启 httpd
$ systemctl restart httpd
# 触发警报以进行测试
$ curl -I http://localhost?exec=/bin/bash
> HTTP/1.1 403 Forbidden
# 测试 PUT
curl -X PUT -I http://localhost
> HTTP/1.1 403 Forbidden

```
## 开放PUT,DELETE 方法

```Bash
$ cd /etc/httpd/crs/rules
$ cp REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
# 从 REQUEST-901-INITIALIZATION.conf 中复制一段修改加到REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf 下
# id 要修改为唯一，在tx.allowed_methods里加PUT DELETE
SecRule &TX:allowed_methods "@eq 0" \
    "id:9011601,\
    phase:1,\
    pass,\
    nolog,\
    ver:'OWASP_CRS/3.3.0',\
    setvar:'tx.allowed_methods=GET PUT DELETE HEAD POST OPTIONS'"
$ apachectl configtest
$ systemctl restart httpd
$ curl -X PUT -I http://localhost
> HTTP/1.1 200 OK
```


## ModSecurity的功能：

1. SQL Injection (SQLi)：阻止SQL注入

   程式寫出的漏洞。 編碼 " 和 ' 可以解決
   
2. Cross Site Scripting (XSS)：阻止跨站脚本攻击

   跨站請求。對程式沒影響，如果有表單使用 CSRF 驗證碼 ，每個表單在打開網址是生成一個用這個網址生成的字串。然後在表單裏用input type=hidden value=驗證碼。 提交後先驗證這個是否正確。


3. Local File Inclusion (LFI)：阻止利用本地文件包含漏洞进行攻击
   
   程式寫出的漏洞， 在使用include,require,include_once,require_once時注意一下變量不是用戶提交的。 

4. Remote File Inclusione(RFI)：阻止利用远程文件包含漏洞进行攻击
 
   程式寫出的漏洞，在使用讀文件時。不要執行讀文件內代碼。例如 include,file_get_contets 遠程文件時。
  
5. Remote Code Execution (RCE)：阻止利用远程命令执行漏洞进行攻击
   
    同樣不執行遠程問題，

6. PHP Code Injectiod：阻止PHP代码注入
   
   主機沒有寫權限就不能執行PHP代碼

7. HTTP Protocol Violations：阻止违反HTTP协议的恶意访问

   使用HTTPS(SSL)

8. HTTPoxy：阻止利用远程代理感染漏洞进行攻击

   主機不使用代理

9. Shellshock：阻止利用Shellshock漏洞进行攻击
    
    限制使用以下function:
    escapeshellarg — 把字符串转码为可以在 shell 命令里使用的参数
    escapeshellcmd — shell 元字符转义
    exec — 执行一个外部程序
    passthru — 执行外部程序并且显示原始输出
    proc_close — 关闭由 proc_open 打开的进程并且返回进程退出码
    proc_get_status — 获取由 proc_open 函数打开的进程的信息
    proc_nice — 修改当前进程的优先级
    proc_open — 执行一个命令，并且打开用来输入/输出的文件指针。
    proc_terminate — 杀除由 proc_open 打开的进程
    shell_exec — 通过 shell 环境执行命令，并且将完整的输出以字符串的方式返回。
    system — 执行外部程序，并且显示输出

    如果有程式使用到的要打開，如果apache用戶沒有權限。即使能執行也不能做什麼，現在我們就是這樣。

10. Session Fixation：阻止利用Session会话ID不变的漏洞进行攻击

   curl 帶 cookie，連續執行帶有會話的請求。 首先要得到SessionID,已經得到了后邊隨便執行什麼了。 

11. Scanner Detection：阻止黑客扫描网站
   
   掃描的主要是網上下載的開源程式。我們沒有。然後是主機端口不關程式。

12. Metadata/Error Leakages：阻止源代码/错误信息泄露

   PHP7 很少。 再有就是程式自己寫出來的錯誤。

13. Project Honey Pot Blacklist：蜜罐项目黑名单
     不知道是啥。

14. GeoIP Country Blocking：根据判断IP地址归属地来进行IP阻断

    限IP  ，我很討厭限制IP, 說明自己有漏洞。只限部分人。 沒見過哪個黑客用自己IP的。
