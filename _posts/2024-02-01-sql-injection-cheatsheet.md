---
title: SQL Injection Cheat Sheet
categories: [ETC., Cheatsheet]
tags: [SQL, Injection]
images:
    path: 
---
SQL Injection 공격 구문 정리를 위한 페이지로 지속적으로 업데이트 예정

## SQL Injection 취약점 진단 포인트


| 취약점 유형                  | 설명                                                                                        |
|-----------------------------|---------------------------------------------------------------------------------------------|
| **입력 폼**                  | 로그인 폼, 검색 폼, 회원 가입 폼 등 사용자 입력이 발생하는 모든 폼이 취약할 수 있습니다. 특히, 아이디와 패스워드 입력란은 주요 대상입니다.     |
| **URL 매개 변수**            | URL에 전달되는 매개 변수는 SQL Injection에 취약할 수 있습니다. 특히, 검색 기능과 관련된 URL 매개 변수는 주요 대상입니다.                  |
| **쿠키**                    | 세션 관리를 위한 쿠키 값은 조작될 경우 보안에 취약할 수 있습니다. 특히, 사용자 인증과 관련된 쿠키 값은 주요 대상입니다.                    |
| **HTTP 헤더**               | 일부 웹 어플리케이션에서는 HTTP 헤더에도 사용자 입력이 삽입될 수 있습니다. 예를 들어, 사용자 에이전트(User-Agent)나 Referer 헤더 등이 그 예입니다. |
| **에러 메시지**              | 어플리케이션이 반환하는 에러 메시지에는 SQL Injection에 대한 정보가 포함될 수 있습니다. 예를 들어, 디버그 모드에서는 SQL 에러 메시지가 노출될 수 있습니다. |
| **다중 질의(Multi-Query)**   | 어플리케이션이 한 번의 요청에 여러 개의 SQL 쿼리를 실행할 경우, 이를 이용하여 SQL Injection을 시도할 수 있습니다.                  |
| **블라인드(SQL Blind)**      | 결과가 화면에 표시되지 않는 경우에도, 조건에 따라 참/거짓 여부를 판단하여 정보를 추출할 수 있는 취약점입니다.                      |
| **저장 프로시저 상호 작용** | 웹 어플리케이션이 저장 프로시저를 사용할 때, 이를 이용하여 SQL Injection을 시도할 수 있습니다.                                |

<br/><br/>


## 공격 구문
```
1' or''='
' OR 1=1 --

```
