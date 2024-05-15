---
title: Lumma Stealer Analysis
categories: [Analysis, Malware]
tags: [malware, analysis, lumma, stealer]
image:
    path: 
---


## [0x00] overview
---
오랜만에 악성코드 분석을 해보고자 lumma stealer 악성코드를 수집집하여 분석해보았다. 쉬운 녀석보다는 조금 어려운 녀석을 분석하고 싶었기에, 파일 크기도 작지 않은 녀석(아마 더미코드가 붙어서 분석이 어렵지 않을까)으로 택했다.

### ananlysis env
- OS: Windows 7 x64
- Platform: Vmware
- C Runtime Libraray: [Link](https://support.microsoft.com/en-us/topic/update-for-universal-c-runtime-in-windows-c0514201-7fe6-95a3-b0a5-287930f3560c)

- Malware
    - md5: 54E97708C9714C69BD34300EA9F397D6
    - file size: 2.96MB

## [0x01] file information
---
기본적인 파일 정보는 아래와 같다. x64, PE, 그리고 exe라는 정보를 알 수 있다.
![](../assets/image_post/20240515164823.png)

함수 정보를 보니 친절하게도 injection 관련 함수가 쉽게 보인다. 
![](../assets/image_post/20240515164945.png)



## [0x02] init process and code
--- 
기본적으로 초반 코드부터 악성 행위를 수행하지는 않고, 문자열 복호화, 코드 복호화 등의 작업을 다수 수행한다. 
![](../assets/image_post/20240515165257.png)

각 함수 사이에도 꽤 적지 않는 내부 함수 호출을 시도하기에 단순히 코드를 순서대로 분석해서 들어가면 오래 걸릴 수 있다.
![](../assets/image_post/20240515165321.png)


## [0x03] process injection
---

필요한 준비가 다 된 후 프로세스를 생성하여 인젝션을 시도한다. 인젝션 대상은 `wab.exe`이다.
> Windows Address Book(wab)는 윈도우 운영 체제에 기본적으로 탑재된 주소록 관리 애플리케이션 

![](../assets/image_post/20240515164153.png)

![](../assets/image_post/20240515165130.png)

이후 ResumeThread를 통해 injection 한 코드가 동작하게끔 한다.
![](../assets/image_post/20240515171524.png)


## [0x04] injected code
---

그렇다면 어떤 코드를 인젝션 하는가 보았더니 MZ 로 시작하는 pe 파일의 내용을 바로 메모리에 맞추어 `WriteProcessMemory`를 사용하고 있었다. 온전한 PE 파일 형태이기에 추출해보았다.

- extraced
    - md5: AA17CF9FF92E097F2B6CF9C993C4ABEC
    - file size: 321KB 

![](../assets/image_post/20240515170653.png)

코드는 많지 않아 보이지만, 마찬가지로 분석하기 어렵게 꼬아놓아져 있는 형태이다.
![](../assets/image_post/20240515170807.png)

동적으로 분석하고 싶었지만, 실행 시 별 다른 행동을 하지 않고 종료된다.


## [0x05] bypass exit 
사용할 코드를 복호화 한다.
![](../assets/image_post/20240515172343.png)

![](../assets/image_post/20240515172421.png)

![](../assets/image_post/20240515172526.png)


sha 알고리즘과 관련된 항목을 찾는다.
![](../assets/image_post/20240515173045.png)




![](../assets/image_post/20240515174055.png)



![](../assets/image_post/20240515174443.png)

정상 동작을 위해 아래와 같이 opcode를 수정해준다.
![](../assets/image_post/20240515175407.png)

## [0x00] reference
---

main
- md5: 54E97708C9714C69BD34300EA9F397D6
- sha256: 59389ead2fa31decb31a25cfbe8d9859d831ef50bc21f9cde1aeb3c074b6d568

extract
- md5: AA17CF9FF92E097F2B6CF9C993C4ABEC
- sha256: 3984a5df1e5178681a26b5f55cf6c293bd88d487e8838a822df4f3ef0c4204d2