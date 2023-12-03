---
title: "[Git] PR(Pull Request) 날리는 법"
date: 2023-10-11T17:26:57+09:00
draft: false
pin: false
summary: 협업할때 Github Pull Request 날리는법
tags:
  - git
---

> # 1. 원본레포지토리 Upstream 으로 설정하기

원본레포지토리는 본인의 git 계정에있는 fork한 레포지토리가아니라 원래의 fork를 뜬 레포지토리를 말한다.

```bash
git remote add upstream [원본 레포지토리 URL] # 준유꺼
git remote -v # 잘됐나 확인
```


> # 2. 개발을 시작하기전에 Branch를 만든다.

```bash
git branch -b [브랜치 이름] # 대괄호쓰는거아님
```

> # 3. 개발을 끝낸 후 원격 저장소에 Push 한다.

```bash
git add .
git commit -m "커밋 메세지"
git push origin [브랜치 이름]
```


> # 4. Git에 접속하여 Pull Request를 한다

![](https://drive.google.com/uc?export=download&id=1Dssv4u28HI22rlBAQlmgKi4bhEMZVaUj)

위의 사진처럼 `Compare & pull request` 버튼이 활성화 된 것을 확인할 수 있습니다.

버튼을 눌러주시면,

![](https://drive.google.com/uc?export=download&id=1nbODBCjvQIiXCvVvD2vznsT3bIVfqspx)

이처럼 기여할 프로젝트에서 미리 설정된 Pull Request 양식이 뜹니다. 그러면 여기에 프로젝트의 양식과 규칙에 맞게 자신이 기여한 것들을 작성해 주시면 됩니다.

그 다음 `Create pull request` 버튼을 눌러서, 오픈소스 프로젝트에서 나의 PR이 승인되어 merge될 때 까지 기다립니다.

> # 5. PR '승인'이 되었다면 branch 삭제하기

```bash
git branch -D [PR이 승인된 브랜치 이름] # 로컬 브랜치 삭제
git push origin :branch_name # 원격 브랜치 삭제
```


> # 참고(중요!!!!)


```항상 개발을 시작하기전에 원본레포지토리와의 동기화를 진행해야한다```

따라서 위의 1번에서 설정한 Upstream으로 부터 항상 pull을 받고 개발을 시작해야한다!

```bash
git pull upstream main
```

