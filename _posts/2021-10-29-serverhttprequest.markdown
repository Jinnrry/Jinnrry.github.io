---
layout:     post
title:      "/dev/tcp实现http请求"
subtitle:   " "
date:       2021-10-29 12:00:00
author:     "Jinnrry"
header-img: "img/26.jpg"
catalog: true
tags:
    - linux
---
> 在服务器没有安装wget和curl的情况下利用/dev/tcp实现http请求



```
#!/bin/bash
function __curl() {
  read proto server path <<<$(echo ${1//// })
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}

  [[ x"${HOST}" == x"${PORT}" ]] && PORT=80

  exec 3<>/dev/tcp/${HOST}/$PORT

  echo -en "GET ${DOC} HTTP/1.0\r\nHost: ${HOST}\r\n\r\n" >&3
  (while read line; do
   [[ "$line" == $'\r' ]] && break
  done && cat) <&3
  exec 3>&-
}



```

### Tips:

/dev/tcp并不是linux下面的一个文件。直接在/dev目录下面去是找不到这个文件的。这仅仅是bash的一个feature，因此这种方法也仅能在bash中使用，换了其他命令终端就没用了