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
