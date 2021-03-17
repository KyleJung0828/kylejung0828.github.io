---
layout: single
title:  "[Python] Fluent Python (전문가를 위한 파이썬) Chapter 4"
date:   2021-08-08
category: python
tags: [Python]
---

> Fluent Python (전문가를 위한 파이썬)의 Chapter 4 부분을 공부하고 정리한 내용입니다.

## 4.6 제대로 비교하기 위해 유니코드 정규화하기

- 앞 문자에 연결되는 발음 구별 기호 (diacritical mark)는 출력할 때 앞 문자와 결합되어 출력된다.

```python
>>> s1 = 'café'
>>> s2 = 'cafe\u0301' ---> (1)
>>> s1, s2
('café', 'café') ---> (2)
>>> len(s1), len(s) ---> (1)
(4,5)
>>> s1 == s2
False
```

(1) cafe 뒤에 발음 구별 기호가 붙어서 길이는 5로 나온다.

(2) 출력해보면 발음 구별 기호가 앞 문자와 붙어 4글자처럼 출력된다.

### 문제

- 유니코드 표준에서는 '`é`'와 `'e\u0301`', 이 두 개의 시퀀스를 '규범적으로 동일하다'라고 하며, 애플리케이션은 이 두 시퀀스를 동일하게 처리해야 한다.
- 하지만 파이썬에서는 이 둘이 동일하지 않다고 판단한다.

### 해결

- `unicodedata.normalize()` 함수가 제공하는 유니코드 정규화를 이용해야 한다.
- 함수의 인수로서 '`NFC`', '`NFD`', '`NFKD`' 중 하나를 사용해야 한다.
- NFC vs. NFD
    - NFC는 코드 포인트를 조합해서 가장 짧은 동일 문자열을 생성.
    - NFD는 조합된 문자를 기본 문자와 별도의 결합문자로 분리.
    - 두 방식 모두 제대로 비교 가능

```python
>>> from unicodedata import normalize
>>> s1 = 'café'
>>> s2 = 'cafe\u0301'
>>> len(s1), len(s2)
(4, 5)
>>> len(normalize('NFC', s1)), len(normalize('NFC', s2))
(4, 4)
>>> len(normalize('NFD', s1)), len(normalize('NFD', s2))
(5, 5)
>>> normalize('NFC', s1) == normalize('NFC', s2)
True
>>> normalize('NFD', s1) == normalize('NFD', s2)
True
```

- NFKC,  NFKD
    - K는 호환성을 나타낸다.
    - 포매팅 손실이 발생하더라도 '선호하는' 형태의 '호환성 분할'로 치환한다.

```python
>>> from unicodedata import normalize, name
>>> half = '½'
>>> normalize('NFKC', half)
'1/2'
>>> four_squared = '4²'
>>> normalize('NFKC', four_squared)
'42'
```

- '1/2'은 '½'에 대한 적절한 치환이지만 '4²'를 '42'로 변환하는 것은 의미를 변환시킨다 (함수가 포맷에 대해 전혀 모르기 때문).
    - 저장 정보를 왜곡할 수는 있지만 검색이나 색인 생성을 위한 편리한 중간 형태를 생성할 수 있다 (1/2인치를 검색할 때 ½인치도 함께 찾아내는 검색 엔진)
    - 다만 영구 저장할 때는 데이터 손실되므로 사용하면 안 된다.

### 4.6.1 케이스 폴딩

- 모든 텍스트를 소문자로 변환하는 연산.
- 파이썬 3.3에 추가된 str.casefold() 메서드를 이용

```python
>>> eszett = 'ß'
>>> eszett.casefold()
'ss'
>>> eszett.lower()
'ß'
>>> eszett.lower() == eszett
True
>>> eszett.casefold() == eszett
False
```

- 파이썬 3.4에서는 casefold와 lower가 서로 다른 문자를 반환하는 코드 포인트가 116개 있다 (유니코드 6.3에서 명명된 110,122개 문자의 0.11%)
- 그래서 언제 사용해야 이득인가? —> 포매팅이 어려운 환경에서도 호환성을 보장되기 때문에 str.casefold()를 사용하라는 뜻인 것 같다.

### 4.6.2 정규화된 텍스트 매칭을 위한 유틸리티 함수

- 중간 정리
    - NFC, NFD는 안전하다.
    - NFC는 최고의 정규화된 형태이다.
    - str.casefold()는 대소문자 구분 없이 문자를 비교할 때 가장 좋은 방법이다.
- 그래서 아래와 같이 nfc_equal()과 fold_equal()함수를 util로 만들어놓고 사용하면 좋다고 한다.

```python
from unicodedata import normalize

def nfc_equal(str1, str2):
    return normalize('NFC', str1) == normalize('NFC', str2)

def fold_equal(str1, str2):
    return (normalize('NFC', str1).casefold() == normalize('NFC', str2).casefold())
```

사용 예시

```python
>>> s1 = 'café'
>>> s2 = 'cafe\u0301'
>>> s1 == s2
False
>>> nfc_equal(s1, s2)
True
>>> fold_equal(s1, s2)
True

>>> s3 = 'Straße'
>>> s4 = 'strasse'
>>> s3 == s4
False
>>> nfc_equal(s3, s4)
False
>>> fold_equal(s3, s4)
True
```

### 4.6.3 극단적인 '정규화': 발음 구별 기호 제거하기

- 발음 구별 기호를 무시하거나 정확히 사용하지 못하는 경우가 있고, 검색 시 발음 구별 기호를 제거하면 URL이 읽기 좋아진다. (e.g., 구글 검색)
    - 예시: São_Paulo에서 발음 구별 기호를 제거하는 경우

```python
http://en.wikipedia.org/wiki/S%C3%A3o_Paulo
http://en.wikipedia.org/wiki/Sao_Paulo
```

- 발음 구별 기호를 모두 제거하려면 다음 함수를 사용한다.

```python
import unicodedata
import string

def shave_marks(txt):
    """발음 구별 기호를 모두 제거한다."""
    norm_txt = unicodedata.normalize('NFD', txt') --> (1)
    shaved = ''.join(c for c in norm_txt if not unicodedata.combining(c)) --> (2)
		return unicodedata.normalize('NFC', shaved) --> (3)
```

(1) 모든 문자를 NFD를 통해 기본 문자와 결합 표시로 분해한다.

(2) 결합 표시를 모두 걸러낸다. (unicodedata.combining() 함수가 결합 문자인지 확인하는 역할)

(3) 문자를 모두 재결합시킨다.

```python
>>> Greek = 'Ζέφυρος, Zéfiro'
>>> shave_marks(Greek)
'Zεφυρος, Zefiro' --> (1)
```

(1) έ와 é가 바뀌었다

- shave_marks() 함수는 단순히 악센트만 제거하기 때문에 아스키 문자로 만들 수 없는 그리스 문자도 변경한다.
    - 그래서 기반 문자를 분석해서 라틴 알파벳인 경우에만 제거하는 방법이 더 좋다. (unicodedata.combining() 함수로 결합 문자인지 확인하고, string.ascii_letters 안에 있는지 체크하여 라틴 문자인지 판단)
- 원형 따옴표, 전각 대시, 작은 점 등 서양 텍스트에서 널리 사용되는 기호들을 아스키 문자로 변환하는 더 극단적인 방법도 있다.
- 상황에 따라서 깊게/얕게 변환할지 판단해야 한다.

## 4.7 유니코드 텍스트 정렬하기

- 파이썬은 시퀀스 안의 항목을 하나하나 비교하므로, 어떠한 자료형의 시퀀스도 정렬할 수 있다.
    - 문자열의 경우에는 코드 포인트를 비교하는데, 비아스키 문자를 사용하는 경우 부적절한 결과가 발생할 수 있다.

    ```python
    >>> fruits = ['caju', 'atemoia', 'cajá', 'açai', 'acerola']
    >> sorted(fruits)
    ['acerola', 'atemoia', 'açai', 'caju', 'cajá']
    ```

    - 포르투갈어 등 라틴 알파벳을 사용하는 언어에서는 정렬할 때 악센트와 갈고리형 기호가 거의 영향을 미치지 않으므로, 'cajá'가 'caju'보다 앞에 나와야 한다.
        - 비슷한 논리로 'açai'가 'acerola' 앞에 나와야 한다.
    - 비아스키 텍스트는 locale.strxfrm() 함수를 이용해서 변환하는 것이 표준이다.
    - 먼저 애플리케이션에 대해 적절히 현지어를 설정하고 OS가 이 설정을 지원하도록 기도해야 한다.

    ```python
    >>> import locale
    >>> locale.setlocale(locale.LC_COLLATE, 'pt_BR.UTF-8') ---> (1)
    'pt_BR.UTF-8'
    >>> fruits = ['caju', 'atemoia', 'cajá', 'açai', 'acerola']
    >>> sorted_fruits = sorted(fruits, key=locale.strxfrm)
    >>> sorted_fruits
    ['açai', 'acerola', 'atemoia', 'cajá', 'caju']
    ```

    (1) 정렬할 때 locale.strxfrm() 함수를 키로 사용하기 전에 setlocale(LC_COLLATE, <지역_언어>)를 호출해야 한다.

    - 주의할 점:
        - 지역 설정은 시스템 전역에 영향을 미치므로 라이브러리의 setlocale()을 호출하지 마라 (프로세스를 시작할 때 지역을 설정하고 그 후에는 변경하면 안 된다).
        - locale 모듈이 OS에 설치되어 있어야 한다 (그렇지 않으면 예외 발생).
        - 지역명의 철자를 제대로 알아야 하므로 MSDN의 '언어 식별자 상수 및 문자열'과 '코드 페이지 식별자' 문서를 잘 보고 입력한다.
        - OS 제작자에 의해 locale이 올바로 구현되어 있어야 한다. (우분투 14.04에서는 성공하고 OS X 매버릭스 10.9에서는 실패했다)

### 4.7.1 유니코드 대조 알고리즘을 이용한 정렬

- PyPI로 제공되는 PyUCA 라이브러리에 해결책이 있다.

```python
>>> import pyuca
>>> coll = pyuca.Collator()
>>> fruits = ['caju', 'atemoia', 'cajá', 'açai', 'acerola']
>>> sorted_fruits = sorted(fruits, key=coll.sort_key)
>>> sorted_fruits
['açai', 'acerola', 'atemoia', 'cajá', 'caju']
```

- 리눅스/OS X/윈도우에서 잘 작동하며, 파이썬 3.x 버전만 지원된다.
- 지역 정보를 고려하지 않기 때문에 정렬 방식을 커스터마이즈하려면 Collate() 생성자에 직접 만든 대조 테이블에 대한 경로를 제공하면 된다. 기본적으로는 아래와 같은 allkeys.txt를 대조 테이블로 사용한다.

```python
0000  ; [.0000.0000.0000] # NULL (in ISO 6429)
0001  ; [.0000.0000.0000] # START OF HEADING (in ISO 6429)
0002  ; [.0000.0000.0000] # START OF TEXT (in ISO 6429)
0003  ; [.0000.0000.0000] # END OF TEXT (in ISO 6429)
0004  ; [.0000.0000.0000] # END OF TRANSMISSION (in ISO 6429)
0005  ; [.0000.0000.0000] # ENQUIRY (in ISO 6429)
0006  ; [.0000.0000.0000] # ACKNOWLEDGE (in ISO 6429)
0007  ; [.0000.0000.0000] # BELL (in ISO 6429)
0008  ; [.0000.0000.0000] # BACKSPACE (in ISO 6429)
000E  ; [.0000.0000.0000] # SHIFT OUT (in ISO 6429)
000F  ; [.0000.0000.0000] # SHIFT IN (in ISO 6429)
```

## 4.8 유니코드 데이터베이스

![](/assets/images/fluent-python-chapter4/ramanujan.png)

- re_dig는 re 모듈을 이용해 r'\d' 정규표현식과 일치하는 문자를 의미. 유니코드를 잘 인식하지 못한다.
- isdig은 char.isdigit() 함수를 호출한 결과
- isnum은 char.isnumeric() 함수를 호출한 결과
- 오른쪽에서 두 번째 칼럼은 unicodedata.numeric(char) 메서드를 호출한 결과로, 타밀 숫자나 로마 숫자를 지원하는 애플리케이션에서 사용하기 좋을 것 같다.

## 4.9 이중 모드 str 및 bytes API

- 표준 라이브러리에는 str이나 bytes 인수를 모두 받으며 자료형에 따라 다르게 작동하는 모듈 (e.g., re와 os 모듈)이 있다.

### 4.9.1 정규 표현식에서의 str과 bytes

- bytes로 정규 표현식을 만들면 \d와 \w같은 패턴은 아스키 문자에만 매칭되지만, str로 이 패턴을 만들면 아스키 이외에 유니코드도 매칭된다.

```python
import re

re_numbers_str = re.compile(r'\d+') ---> (1)
re_words_str = re.compile(r'\w+') ---> (1)
re_numbers_bytes = re.compile(rb'\d+') ---> (2)
re_words_bytes = re.compile(rb'\w+') ---> (2)

text_str = ("Ramanujan saw \u0be7\u0bed\u0be8\u0bef" ---> (3)
            " as 1729 = 1³ + 12³ = 9³ + 10³.") ---> (4)

text_bytes = text_str.encode('utf_8') ---> (5)

print('Text', repr(text_str), sep='\n  ')
print('Numbers')
print('  str  :', re_numbers_str.findall(text_str)) ---> (6)
print('  bytes:', re_numbers_bytes.findall(text_bytes)) ---> 7)
print('Words')
print('  str  :', re_words_str.findall(text_str)) ---> (8)
print('  bytes:', re_words_bytes.findall(text_bytes)) ---> (9)
```

(1) str형 정규 표현식

(2) bytes형 정규 표현식

(3) 타밀 숫자로 1729를 담고 있는 검색할 유니코드

(4) 컴파일 시 앞 문자열에 연결된다.

(5) bytes 정규표현식을 검색하려면 bytes 문자열이 필요하다.

(6) str 패턴 r'\d+'는 타밀과 아스키 숫자에 매칭된다.

(7) bytes 패턴 rb'\d+'는 아스키 숫자에만 매칭된다.

(8) str 패턴 r'\w+'는 문자, 위첨자, 타밀, 아스키 숫자에 매칭된다.

(9) bytes 패턴 rb'\w+'는 문자와 숫자에 대한 아스키 바이트에만 매칭된다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/947c4c3f-4725-4413-9b43-58d8d7d96bca/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/947c4c3f-4725-4413-9b43-58d8d7d96bca/Untitled.png)

- 정규 표현식을 str과 bytes에 사용할 수 있지만 bytes에 정규 표현식을 사용하면 아스키 범위 바깥의 문자들은 숫자와 단어로 처리하지 않는다.

### 4.9.2 OS 모듈 함수에서 str과 bytes

- GNU/리눅스 커널은 유니코드를 모르기 때문에 실제 OS 파일명은 어떠한 인코딩 체계에서도 올바르지 않은 바이트 시퀀스로 구성되어 있으며 str로 디코딩할 수 없다.
    - 다양한 운영체제를 클라이언트로 가지는 파일 서버는 이런 문제가 발생하기 쉽다.

```python
>>> os.listdir('.') ---> (1)
['abc.txt', 'digits-of-π.txt']
>>> os.listdir(b'.') ---> (2)
[b'abc.txt', b'digits-of-\xcf\x80.txt']
```

(1) 두 번째 파일명은 그리스 문자 파이가 들어갔다.

(2) bytes형 인수를 받은 listdir() 함수는 그리스 문자 파이를 UTF-8로 인코딩한 b'\xcf\x80'을 파일명으로 반환한다.

- 파일명이 str인지 bytes인지에 따라 시퀀스를 수작업으로 처리하는 것을 도와주기 위해 os 모듈에는 다음과 같은 인코딩/디코딩 함수가 있다.
    - fsencode(파일명)
        - 파일명이 str형이면 sys.getfilesystemencoding()이 반환한 코덱명을 이용해서 파일명을 bytes로 인코딩한다. 파일명이 bytes형이면 그대로 반환한다.
    - fsdecode(파일명)
        - 파일명이 bytes형이면 sys.getfilesystemencoding()이 반환한 코덱명을 이용해서 파일명을 str형으로 디코딩한다. 파일명이 str형이면 그대로 반환한다.
    - 유닉스 계열 플랫폼에서 이 함수들은 예상치 않은 바이트에서 문제가 발생하는 것을 피하기 위해 surrogateescape 에러 처리기를 사용한다. 윈도우에서는 strict 에러 처리기가 사용된다.

### surrogateescape를 이용해서 깨진 문자 처리하기

- 예상치 못한 bytes나 모르는 인코딩을 처리하기 위해서는 'PEP 383'의 설명처럼 파이썬 3.1에 소개된 이 에러 처리기를 사용한다.
- 이 에러 처리기는 디코딩할 수 없는 바이트를 유니코드 표준에서 '하위 써로게이트 영역'이라고 하는 U+DC00에서 U+DCFF까지의 코드포인트로 치환한다.
    - 이 영역의 코드 포인트에는 문자가 할당되어 있지 않고, 애플리케이션 내부 용도로 사용할 수 있도록 예약되어 있다.
    - 인코딩할 때 이 코드 포인트는 치환된 원래 바이트로 다시 변환된다.

```python
>>> os.listdir('.') ---> (1)
['abc.txt', 'digits-of-π.txt']
>>> os.listdir(b'.') ---> (2)
[b'abc.txt', b'digits-of-\xcf\x80.txt']
>>> pi_name_bytes = os.listdir(b'.')[1] ---> (3)
>>> pi_name_str = pi_name_bytes.decode('ascii', 'surrogateescape') ---> (4)
>>> pi_name_str ---> (5)
'digits-of-\udccf\udc80.txt'
>>> pi_name_str.encode('ascii', 'surrogateescape') ---> (6)
b'digits-of-\xcf\x80.txt'
```

(1) 비아스키 파일명이 있는 디렉터리

(2) 인코딩 방식을 모른 체하면서 파일명을 bytes로 가져오자.

(3) pi_names_bytes는 파이 문자를 포함하는 파일명이다.

(4) 이 파일명을 'surrogateescape'와 함께 'ascii' 코덱을 이용해서 str형으로 디코딩한다.

(5) 비아스키 바이트가 써로게이트 코드 포인트로 치환되므로, '\xcf\x80'은 \udccf\udc80'이 된다.

(6) 다시 아스키 bytes로 인코딩한다. 각 써로게이트 코드 포인트는 치환되었던 원래 바이트로 다시 치환된다.

## 뒷 이야기

### Str은 메모리 안에서 어떻게 표현될까?

- 공식 문서에서는 언급을 피하고 있다.
- 파이썬 3.3 이전에는 코드 포인트마다 16비트 (좁은 빌드), 32비트 (넓은 빌드)를 사용해서 컴파일할 수 있었다.
    - 하지만 좁은 빌드에서는 2바이트를 초과하는 문자는 투명하게 처리할 수 없고
    - 넓은 빌드에서는 문자당 4바이트를 소비하므로 중국어 상형문자 같이 2바이트 안에 들어가는 경우에서는 메모리가 낭비되는 단점으로 작용한다.
- 파이썬 3.3 이후에는 인터프리터가 str 안의 문자를 조사해서 메모리를 효율적으로 사용할 수 있는 메모리 구조를 사용한다.
    - latin1 범위 문자만 있다면 str은 코드 포인트당 1바이트만 사용하고, 그렇지 않다면 2바이트나 4바이트를 사용한다.
    - 파이썬 3에서 int형이 작동하는 방식과 비슷하다 ([PEP 393](https://www.python.org/dev/peps/pep-0393/)에 자세한 내용이 나와있으니 자세한 내용을 알고 싶으면 여길 참고하면 된다)
