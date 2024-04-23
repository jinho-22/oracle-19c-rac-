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

- systemctl stop firewalld
- systemctl disable firewalld
 
- systemctl stop bluetooth
- systemctl disable bluetooth
 
- systemctl stop chronyd
- systemctl disable chronyd
- mv /etc/chrony.conf /etc/chrony.conf.bak
 
- systemctl stop ntpdate
- systemctl disable ntpdate
 
- systemctl stop avahi-daemon.socket
- systemctl disable avahi-daemon.socket
 
- systemctl stop avahi-daemon
- systemctl disable avahi-daemon
 
- systemctl stop libvirtd
- systemctl disable libvirtd
```

```
rpm 설치

- rpm -ivh oracleasm-support-2.1.11-2.el7.x86_64.rpm
- rpm -ivh oracleasmlib-2.0.12-1.el7.x86_64.rpm
```

```
Temp 파일시스템 할당

- vi /etc/fstab

아래 내용 추가
tmpfs                   /dev/shm                 tmpfs   size=7g         0 0

/dev/shm 영역 remount
- mount -o remount /dev/shm

추가 후 확인
- df -h /dev/shm
```

```
추가한 디스크 확인
- fdisk -l

디스크 포멧 후 다시 확인
- fdisk /dev/sdb

- fdisk -l
```

```
Pv Lv 파티 생성

pvcreate /dev/sdb1
vgcreate 19c /dev/sdb1
lvcreate -L 2g -n OCR_VOTE1 19c
lvcreate -L 2g -n OCR_VOTE2 19c
lvcreate -L 2g -n OCR_VOTE3 19c
lvcreate -L 20G -n DATA 19c
```

```
Oracle ASM 설정 및 시작
- oracleasm configure -i

Configuring the Oracle ASM library driver.
 
This will configure the on-boot properties of the Oracle ASM library
driver.  The following questions will determine whether the driver is
loaded on boot and what permissions it will have.  The current values
will be shown in brackets ('[]').  Hitting <ENTER> without typing an
answer will keep that current value.  Ctrl-C will abort.
 
Default user to own the driver interface []: oracle <-- oracle 입력
Default group to own the driver interface []: dba  <-- dba 입력
Start Oracle ASM library driver on boot (y/n) [n]: y <-- y 입력
Scan for Oracle ASM disks on boot (y/n) [y]: y <-- y 입력
Writing Oracle ASM library driver configuration: done
```

```
아래 명령시 /dev/oracleasm 디렉토리가 만들어지고, oracleasm/disks에 라벨링된 디스크가 저장됨
- oracleasm init
```

```
상태 확인
oracleasm status

Checking if ASM is loaded: yes
Checking if /dev/oracleasm is mounted: yes
 
# oracleasm configure
ORACLEASM_ENABLED=true
ORACLEASM_UID=oracle
ORACLEASM_GID=dba
ORACLEASM_SCANBOOT=true
ORACLEASM_SCANORDER=""
ORACLEASM_SCANEXCLUDE=""
ORACLEASM_SCAN_DIRECTORIES=""
ORACLEASM_USE_LOGICAL_BLOCK_SIZE="false"
```

```
공유 디스크 생성

- oracleasm createdisk OCR_VOTE1 /dev/19c/OCR_VOTE1
- oracleasm createdisk OCR_VOTE2 /dev/19c/OCR_VOTE2
- oracleasm createdisk OCR_VOTE3 /dev/19c/OCR_VOTE3
- oracleasm createdisk DATA01 /dev/19c/DATA
```

```
디스크 스캔 후 생성 리스트 확인
- oracleasm scandisks

- oracleasm listdisks
DATA01
OCR_VOTE1
OCR_VOTE2
OCR_VOTE3
4개가 나옴
```

```
디렉토리 생성 및 권한부여

- mkdir -p /oracle/media
- mkdir -p /oracle/app/oracle/product/19c
- mkdir -p /oracle/app/grid/19c
- mkdir -p /oracle/oraInventory
- mkdir -p /oraarch
- chown -R oracle:dba /oracle
- chmod -R 775 /oracle
- chown -R oracle:dba /oraarch
- chmod -R 775 /oraarch
- chown -R oracle:dba /dev/oracleasm
- chown -R oracle:dba /dev/19c
```

```
/oracle/media 경로에 설치파일 업로드

GRID
DB
OPatch
RU
파일을 업로드 후
- chown -R oracle:dba /oracle/media/
입력 
```

```
oracle 계정으로 변경 후
bash_profile에 내용 추가
- vi .bash_profile

export ORACLE_BASE=/oracle/app/oracle;
export ORACLE_HOME=$ORACLE_BASE/product/19c;
export ORACLE_SID=ORADB1;
export GRID_HOME=/oracle/app/grid/19c;
export GRID_SID=+ASM1;
export PATH=$ORACLE_HOME/bin:$GRID_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib;
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib;
export DISPLAY=192.168.137.1:0.0;
 
alias grid='export ORACLE_HOME=$GRID_HOME; export ORACLE_SID=$GRID_SID; export PATH=$ORACLE_HOME/bin:$GRID_HOME/bin:$PATH; echo $ORACLE_SID; echo $ORACLE_HOME'
alias db='. ~oracle/.bash_profile;export PATH=$ORACLE_HOME/bin:$GRID_HOME/bin:$PATH; echo $ORACLE_SID;echo $ORACLE_HOME'
alias oh='cd $ORACLE_HOME;pwd'
alias ss='sqlplus / as sysdba'

. ./.bash_profile
적용 
```

## 이제 노드2 설정
노드1 파일 복사 후 복사한 파일 이름 변경
oel19db1 --> oel19db2


Network Adapter(NAT, Host-only 모두) 선택 후 Advanced 선택
Generate을 눌러서 MAC Address변경

Options 에서 Virtual machine name을 oel19db1에서 oel19db2로 변경

노드2를 root계정으로 로그인 후
IP 변경
```
ens32
- vi /etc/sysconfig/network-scripts/ifcfg-ens32
192.138.137.10 -> 192.138.137.20
 
ens34
- vi /etc/sysconfig/network-scripts/ifcfg-ens34
10.10.10.10 -> 10.10.10.20
```

```
네트워크 재시작 및 확인
- nmcli con down ens32
- nmcli con down ens34
- nmcli con up ens32
- nmcli con up ens34
- systemctl restart NetworkManager.service
- ifconfig
```

```
hostnamectl 명령으로 hostname 변경 oel19db1-> oel19db2 후 재기동

- hostnamectl set-hostname oel19db2
- reboot
```

```
1번노드, 2번노드 모두 OS 기동 후 ping TEST

1번 노드(2번노드의 ip로 ping 시도)
- ping oel19db2
- ping oel19db2-priv
 
2번 노드(1번노드의 ip로 ping 시도)
- ping oel19db1
- ping oel19db1-priv
```

```
2번노드 오라클 계정 설정

- vi .bash_profile
export ORACLE_SID=ORADB2;
export GRID_SID=+ASM2;
두가지만 변경 후 적용
. ./.bash_profile 

```

