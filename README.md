# oracle 19c rac 설치 가이드

## VMware 환경
```
  RAM : 4GB
  Processors : 2
  기본 디스크 : 60GB
  추가 디스크 : 30GB
  Host-only 네트워크 추가
```


```
- disk.locking = "FALSE"
- diskLib.dataCacheMaxSize = "0"
- scsi1.sharedBus = "virtual"
- scsi1:0.deviceType = "disk"
디스크 설정
```

## OS 설정

```
- cat /etc/hostname
호스트 이름 확인 후

- hostnamectl set-hostname oel19db1
- reboot
원하는 호스트 이름 설정 후 부팅
```

## Network 설정(GUI 기준)
설정 버튼 클릭 후
![image](https://github.com/jinho-22/oracle-19c-rac-/assets/129517591/f79ba067-095a-421a-91a5-010ee12255c0)


![image](https://github.com/jinho-22/oracle-19c-rac-/assets/129517591/8ab49bb1-db74-4d12-9a66-1ca5c91fe24c)
ens32와 ens34 변경

### ens32
IPv4로 이동

Address를 192.168.137.10으로 변경 후 Apply


### ens34
Connect automatically, Make available to other users 선택 후 IPv4 선택

Manual 선택 후 Address/Netmask를 10.10.10.10/24 설정 후 Apply


ens32 ens34모두 off로 바꾼 후 다시 on으로 바꿔 IP변경되었나 확인

터미널에서도 확인
```
- ifconfig
```


## 오라클 설치 전 사전설정
```
자동 설정
- yum install -y https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
```

```
- vi /etc/sysctl.conf
상세 내용 변경

fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 2147483648 (물리메모리 크기의 절반(byte)으로 설정하였음)
kernel.shmmax = 2147483648 (물리메모리 크기의 절반(byte)으로 설정하였음)
kernel.panic_on_oops = 1
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
```

```
Shell Limits 설정
- vi /etc/security/limits.d/oracle-database-preinstall-19c.conf

oracle   soft   nofile    1024
oracle   hard   nofile    65536
oracle   soft   nproc    16384
oracle   hard   nproc    16384
oracle   soft   stack    10240
oracle   hard   stack    32768
oracle   hard   memlock    3865470566(HugePage 사용시 물리메모리의 90% 이상)
oracle   soft   memlock    3865470566(HugePage 사용시 물리메모리의 90% 이상)
```

```
유저 및 그룹 생성

- groupadd dba
- useradd -g dba -G dba oracle
```

```
패스워드 설정

passwd oracle
```

```
selinux disable 설정 변경
- vi /etc/selinux/config

SELINUX=disabled
```

```
불필요한 서비스 정지

systemctl stop firewalld
systemctl disable firewalld
 
systemctl stop bluetooth
systemctl disable bluetooth
 
systemctl stop chronyd
systemctl disable chronyd
mv /etc/chrony.conf /etc/chrony.conf.bak
 
systemctl stop ntpdate
systemctl disable ntpdate
 
systemctl stop avahi-daemon.socket
systemctl disable avahi-daemon.socket
 
systemctl stop avahi-daemon
systemctl disable avahi-daemon
 
systemctl stop libvirtd
systemctl disable libvirtd
```

```
rpm 설치

rpm -ivh oracleasm-support-2.1.11-2.el7.x86_64.rpm
rpm -ivh oracleasmlib-2.0.12-1.el7.x86_64.rpm
```
