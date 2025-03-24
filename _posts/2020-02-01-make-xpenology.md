---
title: 헤놀로지 구축기
date: 2020-02-01
categories: [홈서버]
tags: [xpenology]     # TAG names should always be lowercase
---

## 준비물 

### 하드웨어

|   CPU    | i5-3570 <br> (Cooler: THERMOLAB TRINITY)                                                                                     |
|   RAM    | SAMSUNG PC3-12800U(DDR3 1600Mhz) 8GBx2 <br> ~~TAMMUZ PC3-12800U(DDR3 1600Mhz) 8GBx2~~ <br> (혼용 시 부팅 안될때 있어서 제거) |
|   M/B    | ASUS P8H77-V                                                                                                                 |
|   PSU    | COOLMAX FOCUS Series 600W 80Plus 230V EU                                                                                     |
|   LAN    | Intel EXPI9301CT x 2EA                                                                                                       |
|   CASE   | 3RSYS L700 Eclipse                                                                                                           |
|   SSD    | SAMSUNG 840 Series 120GB                                                                                                     |
|   HDD    | WD RED 4TB <br> TOSHIBA 3TB                                                                                                  |

단, CPU는 4세대 이상 권장(내껀 3세대 ㅠ)

램은 16기가까지 필요없음

### 소프트웨어

| Bootloader  | DSM 6.2 Loader for DS3615xs v1.03b <br> (SanDisk Ultra Fit CZ430 16GB) |
| DSM Version | DSM 6.2.2-24922 Update 4                                               |


## 설치순서
1. 부트로더 파일수정
    * 사용할 USB의 vid, pid 입력(장치관리자에서 USB 선택 후 자세히 탭에서 하드웨어 ID)
    * Serial Generator로 DS3615xs 시리얼 획득 후 입력
    * mac1에 첫번째 랜카드 MAC주소 입력(MAC주소 랜카드에 써져있음)
    * mac2 변수 추가하여 나머지 랜카드 MAC주소 입력(갯수만큼 번호늘려서 추가)
    * netif_num값 랜카드 갯수로 수정
    * sata_args의 SataPortMap값 메인보드 사타포트갯수로 수정
    * common_args_3615 매개변수에 방금 수정한 SataPortMap 매개변수 추가
2. 부트로더 설치
    * USB 포맷 안되어있을 경우
    * diskpart 관리자권한으로 실행
    * ```list disk``` 로 디스크 목록 조회 
    * ```sel disk 디스크번호``` 로 디스크 선택
    * ```clean```
    * Win32DiskImager 프로그램으로 수정한 부트로더 파일 USB에 Write
3. 부팅순서 변경
    * USB를 PC에 꽂고 CMOS 설정화면 진입
    * 부팅순서 1순위로 부트로더 USB로 설정하되 uefi 없는거로 설정
    * 부트로더USB 이외에 부팅 비활성화
4. DSM 설치
    * 부팅 전 SSD 이외 HDD 연결해제
    * NAS_ip:5000 접속
    * 수동설치로 DSM pat파일 선택하여 설치
    * 재부팅하고 화면뜨면 성공 ㅊㅊ
