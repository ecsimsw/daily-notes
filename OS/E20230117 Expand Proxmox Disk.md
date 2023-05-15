# Proxmox VM SIZE 조정

### proxmox console resize disk
![Unknown](https://user-images.githubusercontent.com/46060746/213077038-ac403204-0c53-4526-ae69-fbf737fee0a2.png)

### df -hT : 디스크 사용량 확인
![Unknown](https://user-images.githubusercontent.com/46060746/213077119-e4ac0f31-81ba-4c6b-ad79-dce61d5483cf.png)

 
### lsblk : 블럭 장치 목록 출력   
![Unknown](https://user-images.githubusercontent.com/46060746/213077152-af2e3e15-be63-4e18-9480-02e37a80060e.png)

 
### fdisk -l : 파티션 정보 출력   
![Unknown](https://user-images.githubusercontent.com/46060746/213077180-5080e687-5637-439e-a976-0dbc1e032116.png)

/dev/sda (하드티스크)에 30GiB 할당 확인 
(1GB는 1000³바이트, 1GiB는 1024³바이트)
 
### growpart /dev/sda 3 : 파티션 크기 확장
![Unknown](https://user-images.githubusercontent.com/46060746/213077244-e058e7be-fae7-4fa4-9d3a-6974d2d05d23.png)

파티션 /dev/sda3의 크기는 확장되었으나 논리  (ubuntu--vg-ubuntu--lv)의 크기는 그대로 19G
 
### pvs : pv 리스트 확인
![Unknown](https://user-images.githubusercontent.com/46060746/213077283-0ab967ce-9fb1-45e3-8fbd-27fdfc6419eb.png)

### pvresize /dev/sda3 : 해당 파티션의 pv 사이즈 재조정
![Unknown](https://user-images.githubusercontent.com/46060746/213077319-b06612bd-3568-4270-b2c4-37bc451bc6b0.png)

### lvs : lv 리스트 확인
![Unknown](https://user-images.githubusercontent.com/46060746/213077346-4f8730e3-d880-4395-8e24-88527041cbb8.png)

### lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv : lv 사이즈 확장
![Unknown](https://user-images.githubusercontent.com/46060746/213077367-4431301e-da13-400b-b179-0632b4f02490.png)

### resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv : 파일시스템 재정의
![Unknown](https://user-images.githubusercontent.com/46060746/213077387-eb8c971c-01fa-4b03-8d77-66620d850b22.png)

### df -hT : 적용 확인
![Unknown](https://user-images.githubusercontent.com/46060746/213077415-4b2e5ebf-b7b4-456a-9b01-4c4f59cc7382.png)


# LVM (Logical Volume Manager)

PV(물리 볼륨)을 모아 VG(볼륨 그룹)을 만들고, 이를 다시 논리적으로 나눠 LV(논리 볼륨)으로 사용한다. 사용자가 PV나 VG의 하드웨어단이 아닌, 추상화된 LV를 하나의 디스크나 파티션으로 사용할 수 있게 한다.    

이번 작업은 /dev/sda3(PV)의 사이즈를 늘리고 논리 볼륨을 늘린 PV만큼 확장한 것이다. PV를 늘렸다고 이번처럼 꼭 LV을 이 사이즈에 맞게 키울 필요는 없다. 아래 그림처럼 확장된 VG를 다시 다른 LV으로 나눌 수도 있는 것이다.   

단 이번 상황에서는 단일 LV이 필요, 해당 볼륨을 늘려야 했어 PV를 확장(pvresize)하고 그만큼 LV를 확장(lvextend)하였다. 이후 그 볼륨을 관리/사용할 방식을 지정하는 것으로 파일 시스템을 정의(resize2fs)했다.  

![lvm-on-ubuntu](https://user-images.githubusercontent.com/46060746/213082221-a9e55e3e-b7d0-4902-8185-cb8296743927.png)

## REF)
Expand Ubuntu 20 Proxmox Disk 
proxmox resize storage (DiskPressure) 
