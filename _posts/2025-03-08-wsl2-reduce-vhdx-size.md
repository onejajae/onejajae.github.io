---
title: WSL2 사용 시 vhdx 파일 크기 줄이기
date: 2025-03-08
categories: [WSL]
tags: [wsl]
---

## Introduction
WSL2 인스턴스 내에서 파일을 삭제해도 VHDX 파일이 자동으로 축소되지 않는다. 이로 인해 디스크 용량을 과도하게 사용하기 때문에 이를 해결하고자 함.
  
## Solution
* `diskpart`{: .filepath} 유틸리티를 사용하여 크기를 줄일 수 있음.
* 관리자 권한으로 PowerShell 실행
* WSL 실행 중인 경우 종료
``` powershell
wsl --shutdown
```

* `diskpart`{: .filepath} 실행
``` powershell
diskpart
```

* `diskpart`{: .filepath} 에서 VHDX 파일 선택
``` powershell
DISKPART> select vdisk file="VHDX_파일_경로" 
```

* 선택한 VHDX 파일 압축 실행 
``` powershell
DISKPART> compact vdisk
```

## References
1. [How to Shrink a WSL2 Virtual Disk](https://stephenreescarter.net/how-to-shrink-a-wsl2-virtual-disk/)
