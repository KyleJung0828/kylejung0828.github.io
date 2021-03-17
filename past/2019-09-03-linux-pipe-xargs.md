---
title:  "Unix의 redirection, pipe, 그리고 xargs"
date:   2019-09-03
categories: Unix
tags: [Unix, Linux, MacOS, Terminal]
---

### Objectives
Unix 환경 (Linux, MacOS 등)에서의 redirection(`>`, `>>` 또는 `<`), pipe(`|`), 그리고 xargs에 대해 간단히 알아보고, 사용상 유의할 점을 살펴봅니다.

### Pre-requisites
- Unix (or -like) OS: Linux, MacOS, CentOS, <s>TmaxOS</s>...
- Shell (Shell에 대한 이해도가 높지 않다고 가정합니다)

### 들어가기 전에
Redirection과 pipe는 모두 표준 입/출력 스트림 (Standard Input/Output Streams)과 표준 오류 스트림 (Standard Error Streams)을 이용하는 기능입니다. 표준 입/출력과 표준 오류, 그리고 명령어가 어떻게 처리되는지에 대한 사전 지식이 없다면 redirection을 이해하기 조금 힘들 수 있기 때문에 이들에 대해 간단히 살펴보겠습니다.

### Background
Unix는 Shell 환경에서 여러 Command를 지원합니다. 간단한 예를 들어, 현재 디렉토리에 파일 3개와 subdirectory 1개가 있다고 생각해봅시다. 여기서 `ls` 라는 명령어를 치게 되면, 아래와 같이 subdirectory와 file을 모두 list up 해줍니다.
```
$ ls
file1 file2 file3 subDir1
```
그런데 `ls`+`enter` 라고 키보드에서 치면 어떻게 저런 결과가 나오게 되는걸까요? 아래 그림을 설명과 함께 살펴봅시다.
![](/assets/images/2019-09-03/redirection-ipc-depicted.png)
현재 `ls`를 치려고 하는 곳은 Shell이라는 프로그램입니다 (사진에서는 text terminal이라고 표현되었지만, 대충 비슷한 것이라고 이해하면 편합니다). Shell은  키보드를 이용한 입력을 받고, 이 입력을 표준 입력 (stdin)이라고 합니다. 표준 입력은 일련의 과정을 거쳐 `ls`라는 프로그램이 실행될 수 있는 process로 전달되고, 이 process에서 `ls`에서 정의된 동작, 즉 현재 디렉토리의 file과 subdirectory를 나열하는 동작을 수행하고, 그 결과를 Shell이 출력할 수 있도록 스트림이라는 자료로 보관하게 됩니다.

만약 명령이 성공적으로 수행되었다면 표준 출력 (stdout) 스트림으로 Shell에 전달될 것이고, 명령에 오류가 있었다면 표준 오류 (stderr) 스트림으로 전달되어, Shell이 알맞게 출력해주는 것입니다. 이 3가지 입/출력 스트림을 구분하기 위해 다음과 같이 숫자 0,1,2를 사용합니다.
- 표준 입력 (stdin): 0
- 표준 출력 (stdout): 1
- 오류 출력 (stderr): 2

*Shell이 어떻게 동작하는지에 대한 디테일은 [이 포스팅][1]에서 잘 다뤘으니, 읽어보면 좋을 듯 합니다 (개인적으로는 Shell, app, kernel 간의 관계를 충분히 이해하는 개발자와 모르는 개발자는 천지 차이라고 생각합니다).*

### Redirection
Redirection은 `>`, `>>`, 또는 `<`의 기호를 사용하며, redirection이라는 문자 그대로 입/출력에 대한 스트림을 다른 곳으로 돌리거나 전용(轉用)하는 기능이라고 생각하면 됩니다. 백견이 불여일타이기 때문에 예제를 보면서 하나씩 설명하겠습니다.
1. `command > filename` : command 실행의 표준 출력 스트림을 filename이라는 이름을 가진 파일에 redirect합니다.
```
$ ls
file1 file2 file3 subDir1 # 현재 디렉토리의 file과 subdirectory 출력
$ ls > file4 # ls의 결과물을 file4라는 새로운 file에 작성합니다.
$ cat file4 # file4의 내용을 출력합니다.
file1
file2
file3
file4
subDir1
```
한 가지 신기한 점은, `ls > file4` 명령이 실행되었을 때 `file4`가 먼저 만들어지고 `ls` 명령을 통해 `file4`에 쓰여진 것을 보실 수 있습니다. 또, 이 명령이 실행될 때, 명령의 표준 출력 스트림이 `file4`로 redirect되었기 때문에 Shell에는 아무 것도 출력되지 않는 것을 확인할 수 있습니다.
2. `command > /dev/null` : command 실행의 표준 출력 스트림을 버립니다.
```
$ ls > /dev/null
```
아무 것도 출력되지 않습니다. `ls` 명령이 실행되나, 출력 스트림이 `/dev/null`로 redirect되는 것인데, `/dev/null`은 OS에서 흔히 말하는 "Null 장치"로, 여기로 전송된 데이터는 버려집니다. 즉, 특정 명령 실행 후 굳이 출력하고 싶지 않고 버릴 때 사용합니다.
3. `command 2> filename` : command 실행의 표준 오류 스트림을 filename이라는 이름을 가진 file에 redirect합니다.
```
$ la 2> file5
$ cat file5
-bash: la: command not found
```
`ls` 대신 `la` 라는 명령을 잘못 쳤을 경우, 그런 명령이 없다는 표준 오류 스트림이 `file5`에 저장된 결과를 확인할 수 있습니다.
4. `command 2>&1 filename` : command 실행의 표준 오류 스트림을 표준 출력 스트림으로 redirect합니다.
file 이름과 1을 구분하기 위해 앞에 &를 붙입니다.
5. `command 1>&2 filename` : command 실행의 표준 출력 스트림을 표준 오류 스트림으로 redirect합니다.
4번 예시와 마찬가지로 파일 이름과 구분하기 위해 &를 붙입니다.
6. `command >> filename` : command의 실행 결과를 filename이라는 이름을 가진 file 내용에 이어 붙입니다 (append 동작).
```
$ echo "textInFile2" > file2 # "textInFile2"라는 내용을 file2라는 새로운 파일에 작성.
$ cat file2 # file2의 내용물이 제대로 출력된 모습을 확인.
textInFile2
$ ls >> file2 # ls의 결과물을 file2에 이어 붙임.
$ cat file2 # file2에 원래 있었던 내용에 ls 결과물이 이어 붙여져 있는 것을 확인.
textInFile2
file1
file2
file3
file4
subDir1
```
기존에 `file2`에 있었던 내용에 `ls` 결과물을 이어 붙인 모습을 확인할 수 있습니다.
7. `command < filename` : filename이라는 이름을 가진 file을 command에 redirect합니다.
```
$ grep -i "le2" < file4
file2
```
`grep` 명령어로 "le2"라는 텍스트를 포함하는 라인을 `file4`에서 찾는 명령어의 실행 결과입니다 (-i 옵션은 case-insensitive를 나타냅니다). `file4`를 `grep`의 argument로 사용하여 `file4` 내부의 텍스트를 제대로 검색한 결과를 보여줍니다. 원래는 `<`이 필요 없지만, 쉬운 예시를 위해 사용하였습니다. `<`는 현업에서는 거의 본 적이 없고, 사용해본 적도 거의 없으니 참고만 하고 넘어가면 될 것 같습니다.

### Pipe
Pipe는 `|`로 나타내며, 왼쪽 명령의 실행 결과를 오른쪽 방향으로 전달합니다. 마찬가지로, 쉬운 이해를 위해 간단한 예시를 들겠습니다.

현재 디렉토리 안의 파일이 무수히 많다고 가정하고, 그 파일들 중 이름에 `le2`가 들어가는 파일을 찾아 내용을 모두 출력하고 싶은 상황입니다. 어떻게 해야될까요?

우선, 정확한 파일 이름을 알아내야 되는데, 그러려면 `ls`를 이용해 현재 디렉토리에 있는 전체 파일 리스트를 알아내고, 거기에서 `grep`을 이용해 "le2"를 포함한 파일 이름을 알아내야 합니다. 다시 말해서 `ls`의 결과물을 `grep`이 사용할 수 있도록 해줘야 하는데, 이럴 때 사용하는 것이 pipe입니다. 왼 편 명령어 뒤에 pipe를 사용하고 오른 편 명령을 입력하면 왼 편의 실행 결과를 오른 편 명령이 가져다 쓸 수 있게 됩니다.
```
$ ls | grep -i le2
file2
```
위와 같이 명령어를 치게 되면, `ls`의 결과물 (`file1 file2 ... `)이 오른쪽으로 넘어가 `grep`이 사용할 수 있게 됩니다. 저 중 "le2"를 포함하는 결과물은 `file2`밖에 없으니, `file2`를 리턴하게 되고, 이게 우리가 찾던 파일 이름입니다. 이제 이 값을 같은 방법으로 `cat` 명령이 사용할 수 있도록 넘겨주면 됩니다.
```
$ ls | grep -i le2 | cat
file2
```
어딘가 이상합니다. `ls | grep -i le2`의 결과물이 `file2`니까 이를 `cat`에게 넘겨주면 결국 `cat file2`와 같지 않나요?

결론부터 말하자면, 아닙니다. 이 때 등장하는 기능이 바로 `xargs`입니다.

### xargs
위의 예제에서 간과한 점은, pipe로 인해 전달받은 결과물은 반드시 "변수화"가 되지는 않는다는 점입니다. `cat`에게는 왼 편의 파이프로부터 전달받은 결과물이 변수인지 스트링인지 모를 것이고, 이 경우에는 "file2라는 스트링을 출력하라는 건가보다" 라고 처리된 것입니다. 이를 방지하기 위해 xargs를 사용하게 되면, pipe로 인해 전달 받은 결과를 변수화시켜 `cat`에게 넘겨줍니다.
```
$ ls | grep -i le2 | xargs cat
textInFile2
```
이제서야 `cat file2`와 같은 결과가 나온 것을 확인할 수 있습니다.

### Subshell
To be added

[1]:https://medium.com/meatandmachines/what-really-happens-when-you-type-ls-l-in-the-shell-a8914950fd73 "ls-l in the shell"
