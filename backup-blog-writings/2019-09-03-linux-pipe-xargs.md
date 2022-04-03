---
layout: post
title:  "[Dev] Linux의 pipe와 xargs"
date:   2019-09-03
tags: [Dev]
---

### Objectives
Linux의 pipe와 xargs, 사용상 유의할 점을 알아본다.

### Pipe
Pipe는 `|`로 나타내며, 왼 편 명령의 실행 결과를 오른쪽 방향으로 전달한다.
간단하게 예시를 들어보자. `ls`라는 명령어를 치게 되면, 아래 예시와 같이 현재 디렉토리 안에 있는 subdirectory와 file을 모두 list up 해준다.
```
$ ls
file1 file2 file3
```
이 출력 결과를 다른 명령에서 사용하고 싶다면 어떻게 해야 할까? 예를 들어, 현재 디렉토리 안의 파일이 무수히 많다고 가정하고, 그 중 `le2`라는 텍스트가 이름에 포함된 파일의 내용을 모두 출력하고 싶을 때, 어떻게 해야 할까? 우선,`cat` 이라는 명령어를 사용해서 해당 조건에 맞는 파일의 내용을 출력할 수 있을 것이다 (e.g., `cat myFile`). 하지만 "le2"라는 텍스트가 포함된 파일의 이름이 정확히 무엇인지는 알고 있지 않은 상태이다. 이럴 때 사용하는 것이 pipe이다. Pipe는 현재 명령의 실행 결과를 오른쪽 명령이 사용할 수 있도록 넘겨준다. 예를 들어
```
$ ls | grep -i le2
file2
```
위와 같이 명령어를 치게 되면, ls의 결과물 (`file1 file2 file3`)을 오른쪽으로 넘겨주어 grep이 사용할 수 있게 된다 (grep 옆에 있는 인자 `-i`는 case-insensitive를 나타낸다).
저 중 le2를 포함하는 결과물은 file2밖에 없으니, file2를 리턴하게 된다. 이제 이 값을 같은 방법으로 cat에게 넘겨주면 된다.
```
$ ls | grep -i le2 | cat
file2
```

```
$ ls | grep -i le2 | xargs cat
textInFile2
```

pipe 다음에 xargs를 쓰게 되면, pipe로 인해 전달 받은 결과를 "변수화" 시킨다. xargs를 사용하지 않으면 실행 결과 값 자체가 전달된다.
좋은 예시는 아니지만, 설명을 위해 간단한 예제 코드를 보자.
```
$ echo myProgram.out | cat
myProgram.out
```
위 결과는 someCharacters라는 문자열을 프린트하고, 이 결과를 cat한다. Pipe의 쓰임새를 방금 알았다면, 아래 코드도 같은 동작을 해야할 거라고 생각할 것이다.
```
$ cat myProgram.out
textInMyProgram
```
결과를 보면 엄연히 다르다는 것을 알 수 있다. cat은 보통 파일의 내용을 출력해주는 명령어이기 때문에 파일 이름을 인자로 받는다. 첫 번째 코드는 문자열로 인식하였기 때문에 문자열을 프린트해준 것 뿐이고, 두 번째 코드는 파일 이름으로 인식한 것이기 때문에 myProgram.out 이름을 가진 파일을 현재 디렉토리에서 찾아 그 안의 텍스트를 출력하게 된 것이다.
만약 첫 번째 코드를 작성할 때 두 번째 코드처럼 동작하게 하고 싶다면, pipe의 결과물을 cat에 넘겨줄 때 문자열이 아니라 변수로 넘겨줘야 할 것이다. 이 때 사용하는 명령어가 xargs 이다.
```
$ echo myProgram.out | xargs cat
textInMyProgram
```
xargs를 사용하고자 하는 명령어 앞에 붙여주게 되면, 해당 명령어가 pipe 결과물을 변수로 인식하게 하기 때문에 두 번재 코드와 같은 동작을 하게 된다.
