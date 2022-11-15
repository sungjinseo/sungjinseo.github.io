---
layout: post
title: "Java의 CompletableFuture란?"
author: "sungjin seo"
date: 2022-11-15 17:43:00 +0900
categories: [Job, Study]
tags: [Java, Async]
comments: true
pin: false
---

### Future
- 비동기적인 작업의 현재 상태를 조회하거나 결과를 가져올 수 있다.

### CompletableFuture
1. 개요
- 자바에서 비동기(Asynchronous)프로그래밍을 가능하게하는 인터페이스.
- Future의 제약사항들을 해결한다.

2. Future 제약
- 예외 처리용 API를 제공하지 않는다.
- 여러 Future를 조합할 수 없다. (ex: Event정보를 받아 다음 Event에 참석할 회원목록조회)
- Future를 외부에서 완료시킬 수 없다. 취소하거나, get()에 타임아웃을 설정할 수는 있다.
- get()을 호출하기 전까지는 future를 다룰 수 없다.
