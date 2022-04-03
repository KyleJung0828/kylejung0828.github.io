---
layout: post
title:  "[dev] std::numeric_limits<T>::max()/min() Error in Windows"
date:   2019-08-14
tags: [dev]
---

I am currently working on building linux-based codes in Windows system, using Visual Studio 12 (kind of old version though). While doing those jobs, I came across some tedious platform dependent issues that I think are worth noting. Let's see the following code block.
```cpp
std::numeric_limits<int>::max();
```
So I was compiling these lines, and the compiler emitted an error saying `error C2059: syntax error : '::'`

원인: <windows.h> 에 다음과 같은 macro definition이 존재함.

```
#define max(a,b) (((a) > (b)) ? (a) : (b))
```
따라서 preprocessor가 std::numeric_limits<int>::max()를 std::numeric_limits<int>::(((a) > (b)) ? (a) : (b))로 치환해버림. 결국 이 구문은 syntax가 맞지 않기 때문에 error로 인식되는 것.
해결1: 가장 좋은 해결 방법은 Compile flag를 이용하는 방법임
```
#define NOMINMAX
```
다만 이 경우, min()이나 max() 함수를 정의하지 않도록 막기 때문에, 다른 windows 관련 api 중 이들을 사용하는 곳이 있다면 오류를 발생할 것임.
해결2: macro expansion을 방지하기 위해 괄호로 감싸는 방법
```
size_t maxValue_ = (std::numeric_limits<size_t>::max)()
//                 ^                                ^
```
이렇게 되면 macro로 인식되지 않기 때문에 컴파일이 잘 됨.

두 해결책 중 상황에 알맞게 사용하여 해결하면 됨.

### Preprocessor
1. Stringizing operator (#)
```
#include <Turboc.h>
#define result(exp) printf(#exp"=%d\n",exp);
void main()
{
     result(5*3);
     result(2*(3+1));
}
```
2. Merge Operator (##)

```
#include <Turboc.h>
#define var(a,b) (a##b)
void main()
{
     int var(Total, Score);
     TotalScore=256;
     printf("총점 = %d\n",TotalScore);
}
```

- Reference
https://stackoverflow.com/questions/27442885/syntax-error-with-stdnumeric-limitsmax
