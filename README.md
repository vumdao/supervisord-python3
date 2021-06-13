<p align="center">
  <a href="https://dev.to/vumdao">
    <img alt="Docker & supervisord & python3" src="https://github.com/vumdao/supervisord-python3/blob/master/img/cover.jpg?raw=true" width="700" />
  </a>
</p>
<h1 align="center">
  <div><b>Docker & supervisord & python3</b></div>
</h1>

## **When migrating application from python2 to python3, I figured out that `supervisord` with `python3 buster` still use python2 to start its process. Check your container of python3 application which uses supervisord to handle background process and READ MORE ⤵️⤵️⤵️**

```
# ps -ef 
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 03:40 ?        00:00:00 /bin/bash /entrypoint.sh
root        10     1  0 03:40 ?        00:00:08 /usr/bin/python2 /usr/bin/supervisord --nodaemon --configuration /etc/supervisord.conf
```

---

## What’s In This Document
- [Check OS and supervisor package in the container](#-Check-OS-and-supervisor-package-in-the-container)
- [Understand and Solve the issue](#-Understand-and-Solve-the-issue)
- [Conclusion](#-Conclusion)

---

### 🚀 **[Check OS and supervisor package in the container](#-Check-OS-and-supervisor-package-in-the-container)**
- Supervisord is a process control system, designed to monitor and control processes. It does not aim to replace init, instead it encapsulates processes inside its own framework, and can start them at boot time, just like we want. There is no reason to go into any more depth about the software at this point.

- Get into the container and run commands

```
root@app-69b9df66d4-jxmnm:/opt/devops/# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

root@app-69b9df66d4-jxmnm:/opt/devops/# apt-cache show supervisor
Package: supervisor
Version: 3.3.5-1
Installed-Size: 1447
Maintainer: Python Applications Packaging Team <python-apps-team@lists.alioth.debian.org>
Architecture: all
Depends: lsb-base, python-pkg-resources, python-meld3, python:any (<< 2.8), python:any (>= 2.7~)
Suggests: supervisor-doc
Description: System for controlling process state
Description-md5: 965223e7558e3d49e112406ca88bda2b
Homepage: http://supervisord.org/
Tag: implemented-in::python, role::program
Section: admin
Priority: optional
Filename: pool/main/s/supervisor/supervisor_3.3.5-1_all.deb
Size: 284260
MD5sum: 4a40c0ed0774b8b43c5645967129eeb1
SHA256: 9f68f3559eac10840d54301d56cca6ba8f7e202398b2f24bbf8e71a2a59785cb
```

**- supervisord version**
```
supervisord -v
3.3.5
```

**- supervisord.conf**
```
[supervisord]
nodaemon=true
logfile = /var/log/supervisor/supervisord.log

[program:app]
command=python -u /opt/devops/app.py
stdout_logfile=/var/log/app.log
autorestart=unexpected
redirect_stderr=true

[program:cron]
autorestart=false
command=cron -f
```

### 🚀 **[Understand and Solve the issue](#-Understand-and-Solve-the-issue)**
- Issue is with the apt-get package for Debian 10 which is base OS for the python3 and the base docker image `python:3.9.1-buster` comes with supervisor 3.3.5 rather than latest supervisor 4.2.2 So seems that is what issue is.

- Snippet of `Dockerfile`

```
FROM python:3.9.1-buster

RUN apt-get update && \
    apt-get install -y \
    supervisor \
```

- We can use the base image from `python:alpine` instead that that comes with supervisor 4.2.2 but if you still prefer using slim-buster, update the `Dockerfile`

```
FROM python:3.9-slim-buster


RUN apt-get update && \
    apt-get install -y \
    vim \
    curl \
    inetutils-ping \
    procps \
    cron

RUN python -m pip install --upgrade pip
RUN pip install supervisor
```

-  Building image

```
Step 4/17 : RUN pip install supervisor
 ---> Running in c783a4c38aab
Collecting supervisor
  Downloading supervisor-4.2.2-py2.py3-none-any.whl (743 kB)
Installing collected packages: supervisor
Successfully installed supervisor-4.2.2 urllib3-1.26.5
```

- Start the container

```
# ps -ef 
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 05:13 ?        00:00:00 /bin/bash /entrypoint.sh
root         9     1  0 05:13 ?        00:00:00 /usr/local/bin/python /usr/local/bin/supervisord --nodaemon --configuration /etc/supervisord.conf

# python -V
Python 3.9.5
```

### 🚀 **[Conclusion](#-Conclusion)**
- Why should not user python:alpine base image, read [here](https://pythonspeed.com/articles/base-image-python-docker-images/)
- In the future, I think `python3:buster` will update the supervisord version but now here is the solution for reference.

---


<h3 align="center">
  <a href="https://dev.to/vumdao">:stars: Blog</a>
  <span> · </span>
  <a href="https://github.com/vumdao/aws-eks-the-hard-way">Github</a>
  <span> · </span>
  <a href="https://stackoverflow.com/users/11430272/vumdao">stackoverflow</a>
  <span> · </span>
  <a href="https://www.linkedin.com/in/vu-dao-9280ab43/">Linkedin</a>
  <span> · </span>
  <a href="https://www.linkedin.com/groups/12488649/">Group</a>
  <span> · </span>
  <a href="https://www.facebook.com/CloudOpz-104917804863956">Page</a>
  <span> · </span>
  <a href="https://twitter.com/VuDao81124667">Twitter :stars:</a>
</h3>
