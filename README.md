# Ubuntu 20.04: Add Swap Space

## 소개
Ubuntu 20.04 리눅스 서버의 메모리 부족으로 인한 오류를 방지하기 위한 방법으로 Swap을 추가하는 방법

## Swap이란?
운영 체제가 더 이상 RAM에 보관할 수 없는 데이터를 임시로 하드 디스크에 저장하기 위해 설정한 스토리지이며, RAM 보다는 훨씬 느리지만 메모리 부족에 대한 대체 방법임

## 1. Swap 정보 확인: swapon

시스템에 사용 가능한 Swap 공간이 있는지 확인하는 방법
```
$ sudo swapon --show
```
출력이 없으면 시스템에 사용 가능한 스왑 공간이 없음을 의미함

## 2. Swap 정보 확인: free

free 유틸리티를 사용하여 활성 스왑 공간을 확일할 수 있음
```
$ free -h
              total        used        free      shared  buff/cache   available
Mem:          964Mi       755Mi        58Mi       1.0Mi       149Mi        58Mi
Swap:            0B          0B          0B
```

## 3. 하드 디스크의 사용 가능한 공간 확인: df

스왑 파일을 만들기 위한 디스크 공간이 있는지 확인   
OCI Ubuntu 20.04 VM에서 확인한 내용임   
```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            445M     0  445M   0% /dev
tmpfs            97M  1.3M   96M   2% /run
/dev/sda1        45G  4.7G   41G  11% /
tmpfs           483M     0  483M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/loop0       56M   56M     0 100% /snap/core18/2632
/dev/loop2       64M   64M     0 100% /snap/core20/1695
/dev/loop1       56M   56M     0 100% /snap/core18/2566
/dev/loop3       64M   64M     0 100% /snap/core20/1623
/dev/loop5       68M   68M     0 100% /snap/lxd/22753
/dev/loop4       92M   92M     0 100% /snap/lxd/23991
/dev/loop6       60M   60M     0 100% /snap/oracle-cloud-agent/44
/dev/loop7       51M   51M     0 100% /snap/oracle-cloud-agent/46
/dev/loop9       48M   48M     0 100% /snap/snapd/17336
/dev/loop8       50M   50M     0 100% /snap/snapd/17576
/dev/sda15      105M  5.2M  100M   5% /boot/efi
tmpfs            97M     0   97M   0% /run/user/1001
overlay          45G  4.7G   41G  11% /var/lib/docker/overlay2/8edfbbde0bf645f71188569d32172647289e6784b362bffdb6507861df81b32e/merged
overlay          45G  4.7G   41G  11% /var/lib/docker/overlay2/abb0a8fe859e874c21da8e2742bc2fc04d8650e4da13680aa7c1edad99161508/merged
overlay          45G  4.7G   41G  11% /var/lib/docker/overlay2/352086cab8c32af1920a63f20bfd060d05e843b6cb18df9ea669ceff4d5ab03a/merged
shm              64M     0   64M   0% /var/lib/docker/containers/77dd98802c535c10da830b26961256f2367efa7b7b2f11312e7580f873f0d833/mounts/shm
shm              64M     0   64M   0% /var/lib/docker/containers/8e67de11fd08794a500e0c8f74cadc93b5de09b3eecce7bee79e1bd55362b974/mounts/shm
shm              64M     0   64M   0% /var/lib/docker/containers/dd29b3abc15860fa7e85ad1bccacc6834e5379dc7fda399963a6a71c426253e0/mounts/shm
```
/ 파티션에 41G 여유공간을 확인할 수 있음

## 4. 스왑 파일 생성: fallocate

2G 스왑 파일 생성 예제
```
$ sudo fallocate -l 2G /swapfile

$ ls -l /swapfile
-rw-r--r-- 1 root root 2147483648 Nov 26 06:51 /swapfile
```

## 5. 스왑 파일 활성화: mkswap

스왑 파일을 스왑 공간으로 전환
```
$ sudo chmod 600 /swapfile

$ sudo mkswap /swapfile
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=49203313-edb2-424e-b351-3f6215ce67d9

$ sudo swapon /swapfile
```

## 6. 스왑 확인

```
$ swapon --show
NAME      TYPE SIZE   USED PRIO
/swapfile file   2G 113.6M   -2

$ free -h
              total        used        free      shared  buff/cache   available
Mem:          964Mi       666Mi        61Mi       1.0Mi       235Mi       148Mi
Swap:         2.0Gi       153Mi       1.8Gi
```

## 7. 스왑 파일 자동 마운트 설정
```
$ sudo cp /etc/fstab /etc/fstab.bak

$ echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
/swapfile none swap sw 0 0

```

## 8. 스왑 설정 조정

스왑을 처리할 때 시스템 성능에 영향을 줄 수 있는 몇 가지 옵션 구성

### 교환 속성 조정: swappiness

시스템이 RAM에서 스왑 공간으로 데이터를 스왑하는 빈도. 0 ~ 100%

```
$ cat /proc/sys/vm/swappiness
60

$ sudo sysctl vm.swappiness=10
vm.swappiness = 10

$ echo 'vm.swappiness = 10' | sudo tee -a /etc/sysctl.conf
vm.swappiness = 10

```

### 캐시 압력 설정 조정: vfs_cache_pressure

시스템이 다른 데이터에 대해 inode 및 dentry 정보를 캐시하도록 선택하는 양을 구성

```
$ cat /proc/sys/vm/vfs_cache_pressure
100

$ sudo sysctl vm.vfs_cache_pressure=50
m.vfs_cache_pressure = 50

$ echo 'vm.vfs_cache_pressure = 50'  | sudo tee -a /etc/sysctl.conf
m.vfs_cache_pressure = 50
```

***
## 참고 링크
- How To Add Swap Space on Ubuntu 20.04: https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04
