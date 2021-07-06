
# **Linux 파티션 생성 후 파일 시스템 만들기**
ref: https://jeongyd.tistory.com/6

1. 디스크 확인 
  -  
    - 디스크에 파티션을 할당할 공간이 있는지 확인
      -  

          fdisk -l

    - 예시

          /dev/xvdc	300 GB

2. xvdc에 파티션을 생성
  -  
    - 디스크 파티션 유틸리티에 접속
      -  
          fdisk /dev/xvdc


3. fdisk Commnad
  -  
    - 순서대로 
      -  
          m : help

          n : new partition

          p : primary

          default

          t : type 

          8e : Linux LVM

          p : print (확인)

          w : write

4. 파일시스템 생성
  -  
    - xfs 로
      -  
          mkfs.xfs -f /dev/xvdc

5. 파일시스템 mount
  -  
    - directory 생성 후 mount
      -  

          mkdir /data

          mount -t xfs /dev/xvdc /data
  -
    - 확인
      -

          df -T

6. 재부팅시 자동 mount setting 
  -
    - 정보 추가
      - 

	        vi /etc/fstab

        	---
        	/dev/xvdc	/data	xfs	defaults	0	0
	        ---

7. umount & 다시 mount
  -
    -
      -

          df -h

          umount /dev/xvdc

          df -h

          mount -a

          df -h