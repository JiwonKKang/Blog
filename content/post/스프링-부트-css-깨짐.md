---
title: "SpringBoot css 깨짐현상"
date: 2023-08-10T20:12:58+09:00
draft: false
pin: true
summary: "스프링부트 css 깨짐현상"
tags: [spring, error]
---

# 스프링부트 css가 깨지는 현상

### 원인

스프링부트는 설정에서 static 폴더가 기본경로로 지정되어있다.
![](https://velog.velcdn.com/images%2Fdev-jih%2Fpost%2Ffb6d81b0-5592-4886-bc38-48c0efa14386%2Fimg3.png)


## 문제해결

따라서 static 밑의 경로만 써주면된다.

```html
<script src="../static/js/scripts.js"></script>
```

이 경로를

```HTML
<script src="/js/scripts.js"></script>
```

이렇게 바꿔주면 된다.