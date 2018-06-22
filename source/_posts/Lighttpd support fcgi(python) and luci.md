---
title: Lighttpd Support FastCGI (python) and LUCI(OpenWrt)
date: 2017-06-20 10:56:27
tags:
	- webserver
categories: openwrt
---

# 1.Backgroud  
We need to use lighttpd as webserver to servive two web page:  

- FastCGI (python bottle web micro framework)  
- LUCI (openwrt's cgi-bin/luci, which is CGI)


# 2. Support LUCI
After we study OpenWrt wiki docs, we know how to make lighttpd support luci.
Two key steps:  

- enable lighttpd's "mod_cgi" module  
- tell lighttpd all requests to "cgi-bin/luci" will send to lua, lighttpd ignore them

``` bash
# Ensure we have to enable mod_cgi, which is in conf.d
# server.modules += ( "mod_cgi" )
#
# detail see openwrt org doc: https://wiki.openwrt.org/doc/howto/luci.on.lighttpd
# 
# wang.kai@sunmedia.com.cn

# tell lighttpd to process requests using lua
cgi.assign += ( "cgi-bin/luci" => "" ) 
```
Detail see OpenWrt Wiki about [LuCI on lighttpd](https://wiki.openwrt.org/doc/howto/luci.on.lighttpd?s[]=lighttpd)

<!-- more -->

# 3. Support FastCGI
The lighttpd fastcgi configuration like this:

``` bash
# Ensure mod_fastcgi mod_rewrite  have been enabled, some modules will be enabled
# in conf.d, BSP3's configuration will have effect on this.
#
# Server.modules += ( 
#		"mod_fastcgi",
#		"mod_rewrite",
# 		)
# wang.kai@sunmedia.com.cn

var.webapp_path = "/usr/share/httpstation"
var.webapp_fcgi = "main.py"

fastcgi.server = (
	"/python_entry" => (
		"icatchtek" => (
			"bin-path" => webapp_path + "/" + webapp_fcgi,
			"socket" => "/tmp/httpstation-fcgi.socket",
			"max-procs" => 1,
			"check-local" => "disable",
		)
	),
)

# [Note]
# Python request will send to "/python_entry" + "$1", except:
# 1. document.domain + "/cgi-bin/luci"
# 2. document.domain + "/luci-static"
url.rewrite-once = (
	"^(/(?!(cgi|luci-static)).*)" => "/python_entry" + "$1",
)
```

## 3.1. fcgi server  
We need to tell lighttpd:  

- "bin-path"     : where lighttpd will startup executable file
- "socket"       : fast cgi use socket send/receive msg
- "check-local"  : Whether check file is existed in "server.document-root"
- "max-procs"    : lighttpd support multiple process (optional)  

Lighttpd docs about FastCGI <http://redmine.lighttpd.net/projects/lighttpd/wiki/Docs_ModFastCGI>

## 3.2. URL Redirect
Because lighttpd support two web pages, we need to do something make url access ok.  
### 3.2.1. Bottle Python Redirect
If fastcgi server work well, we need to add "/python_entry". For example, we access "172.28.52.152/home" should be "172.28.52.152/python_entry". We use "mod_rewrite" module to avoid this embarrassed situation. This module will pre-process URL before we really access it.   

### 3.2.2. LUCI Redirect
We need to keep luci access like "cgi-bin/luci" and "luci-static"(which is used for js, css, etc resource files). So we use:  
``` bash
# [Note]
# Python request will send to "/python_entry" + "$1", except:
# 1. document.domain + "/cgi-bin/luci"
# 2. document.domain + "/luci-static"
url.rewrite-once = (
	"^(/(?!(cgi|luci-static)).*)" => "/python_entry" + "$1",
)
```

*** How to represent "except xxx" ?***  
In Reluar Expression, we use "**?!**" to express except. So next regular expression is except begin with "cgi" or "luci-staic".
>^(/(?!(cgi|luci-static)).*)


