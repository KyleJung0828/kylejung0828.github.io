---
layout: post
title:  "[Dev] Writing/Reading a simple file in C++"
date:   2019-08-26
tags: [dev]
---

```cpp
#include <fstream> // File Stream

std::string filePath = "u8test.txt"; // Current Directory
std::ofstream writeFile(filePath.data());
if (writeFile.is_open())
{
	writeFile << "Hello World!";
	writeFile.close();
}

// Reading
std::ifstream openFile(filePath.data());
if (openFile.is_open())
{
	std::string line;
	while (std::getline(openFile, line))
	{
		std::cout << line << std::endl;
	}
	openFile.close();
}
```


임의의 한글 텍스트 입력 후 저장 (자동으로 UTF-8로 파일 포맷이 지정됨).
touch u8test.txt && vim u8test.txt
현재 인코딩 확인
$ file -bi u8test.txt
text/plain; charset=utf-8
iconv를 이용해서 u16test.txt라는 UTF16LE 파일로 변환
( printf "\xff\xfe" ; iconv -f utf-8 -t utf-16le u8test.txt ) > u16test.txt
변환된 파일의 인코딩 확인
$ file -bi u16test.txt
text/plain; charset=utf-16le
