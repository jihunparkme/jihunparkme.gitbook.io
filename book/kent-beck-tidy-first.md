# Kent Beck Tidy First?

[켄트 백의 Tidy First?](https://product.kyobobook.co.kr/detail/S000212999739) 책을 요약한 내용입니다.

<figure><img src="../.gitbook/assets/kent-beck-tidy-first.jpg" alt=""><figcaption></figcaption></figure>

# PART 01. 코드 정리법

카탈로그는 코드 변경을 위해 지저분한 코드를 마주칠 때마다 적용할 수 있는 작은 설계 움직임을 다룬다.

---

**CHAPTER 01. 보호 구문**

```java
if (조건)
    코드

...

if (조건)
    if (조건)
        코드
```

중첩된 조건은 헷갈리다. 이러한 코드는 정리할 수 있다.

```java
if (조건) return
if (조건) return
코드
```

코드의 세부 사항을 살펴보기 전 염두에 두어야 할 몇 가지 전체 조건이 있다는 것을 말하는 것처럼 보인다.

항상 그리고 반드시 작은 단계를 거쳐 코드를 정리하자.

---

**CHAPTER 02. 안 쓰는 코드**

안 쓰는 코드는 지워 버리세요.

실행되지 않는 코드라면 그냥 지우세요.

많은 코드가 있고, 당장 사용하진 않지만, 미래에 사용하길 원하고, 원래 작성된 방식과 정확히 동일한 방식으로, 여전히 작동하는 경우라면 다시 가져옹ㄹ 수 있습니다.

복구를 비교적 쉽게 하기 위해 각 정리 과정에서 코드를 `조금만` 삭제하세요.