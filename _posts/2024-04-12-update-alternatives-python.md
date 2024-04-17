---
title: python3 update-alternatives
categories: [ETC., Tools]
tags: [python, update_alternatives]
image:
    path: 
---

## [0x00] code
---
``` bash
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1
```

## [0x01] contents
---
- ubuntu 환경 구성 후 python3는 입력되지만, python은 입력되지 않을 때 위 명령어를 통해 등록 후 `python` 실행 시 정상적으로 실행 가능
- 만약 python이 여러 버전으로 설치되어 있는 경우 `which python`과 같은 방식으로 원하는 python 경로 확인 필요

