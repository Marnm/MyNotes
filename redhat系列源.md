---
title: redhat系列源
tags: []
---

清华源 https://mirrors.tuna.tsinghua.edu.cn/

**rhel 7 epel.repo**
```ini
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=http://download.example/pub/epel/7/$basearch
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch&infra=$infra&content=$contentdir
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=http://download.example/pub/epel/7/$basearch/debug
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch&infra=$infra&content=$contentdir
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place it's address here.
#baseurl=http://download.example/pub/epel/7/source/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch&infra=$infra&content=$contentdir
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
```

**CentOS 7 epel.repo**
```ini
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=http://download.example/pub/epel/7/$basearch
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch&infra=$infra&content=$contentdir
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=http://download.example/pub/epel/7/$basearch/debug
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch&infra=$infra&content=$contentdir
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place it's address here.
#baseurl=http://download.example/pub/epel/7/source/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch&infra=$infra&content=$contentdir
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
```

**替换**

```shell
sed -e 's!^metalink=!#metalink=!g' \
    -e 's!^#baseurl=!baseurl=!g' \
    -e 's!//download\.fedoraproject\.org/pub!//192.168.73.3!g' \
    -e 's!//download\.example/pub!//192.168.73.3!g' \
    -e 's!https://mirrors!http://mirrors!g' \
    -i /etc/yum.repos.d/epel*.repo
```
