---
title: "📚 2. Commit Convention"
description: "Git을 통한 협업에서 Commit Convention에 대해 배워봅시다."
date: 2024-01-02
update: 2024-01-02
tags:
  - Git
  - Github
  - Convention
series: "Git & Github 톺아보기"
---

##  Git Commit Convention

Commit Convention은 간단하게 말해 `협업할 때 Commit Message에 대한 서로간의 약속`입니다.

### Commit Structure

커밋 메시지에 대한 약속.
커밋 메시지 구조는 크게 3가지로 나뉩니다 (제목, 본문, 꼬리말)
```
type: Subject -> 제목  
(한칸 띄우기)  
body(생략 가능) -> 본문  
(한칸 띄우기)  
footer(생략 가능) -> 꼬리말 
``` 

각 커밋 메시지 구조에는 규칙이 존재합니다.

### Commit Type

|Type|설명|
| -- | -- |
| Feat: | 새로운 기능 추가 |
| Fix: | 버그 수정 또는 type |
| Refactor: | 프로덕션 코드 리팩토링, 새로운 기능이나 버그 수정 없이 현재 구현을 개선한 경우 |
| !HOTFIX | 급하게 치명적인 버그를 고쳐야 하는 경우 |
| Design: | CSS 등 사용자 UI 디자인 변경 |
| Comment: | 필요한 주석 추가 및 변경 |
| Docs: | 문서를 수정한 경우 |
| Style: | 코드 포맷팅, 세미콜론 누락, 코드 변경이 없는 경우 |
| Test: | 테스트 (테스트 코드 추가, 수정, 삭제, 비즈니스 로직에 변경이 없는 경우) |
| Chore: | 위에 걸리지 않는 기타 변경 사항(빌드 스크립트 수정, asset image, 패키지 매니저 등) |
| Init: | 프로젝트 초기 생성 |
| Rename: | 파일 혹은 폴더명 수정하거나 옮기는 경우 |
| Remove: | 파일을 삭제하는 작업만 수행하는 경우 |

### Subject Rule

1. 제목은 최대 50글자 넘지 않기
2. 마침표 및 특수기호 사용x
3. 첫 글자 대문자, 명령문 사용
4. 개조식 구문으로 작성(간결하고 요점적인 서술)

### Body Rule
1. 한 줄당 72자 내로 작성
2. 최대한 상세히 작성
3. 어떻게 보다는 '무엇을', '왜' 변경했는지에 대해 작성

### Footer Rule
1. 유형: #이슈 번호의 형식으로 작성
2. 이슈 트래커 ID를 작성
3. 여러개의 이슈 번호는 ,로 구분
4. 이슈 트래커 유형은 아래와 같다

## 커밋 예시

signin, signup 기능이 추가되었다고 가정해보면,
커밋 메시지는 아래와 같을 것입니다.

```
Feat: Add signin, signup  
  
회원가입 기능, 로그인 기능 추가(예시를 위해 간단히 작성)  

Resolves: #1
```