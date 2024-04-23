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
```
IPv4로 이동

Address를 192.168.137.10으로 변경 후 Apply
```

### ens34
```
Connect automatically, Make available to other users 선택 후 IPv4 선택

Manual 선택 후 Address/Netmask를 10.10.10.10/24 설정 후 Apply
```

