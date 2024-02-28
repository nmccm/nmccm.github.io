---
title: "파이썬(Python) 활용한 웹사이트 크롤링"
categories:
- Python
tags:
- Crawler
---

- vscode (VisualCode) 다운로드 후 설치를 진행한다. 
- 파이썬 다운로드 후 설치를 진행한다. (python-3.12.1-amd64.exe) 
- 파이썬은 기본 설치가 아닌 "Customize installation", 아래 옵션은 반드시 체크
- "Use admin privileges when installing py.exe" 옵션 체크
- "Add python.exe to PATH" 옵션 체크
- "Optional Features" 화면에 나오는 모든 옵션 체크하고 "Next" 선택
- "Advanced Options" 화면에 나오는 모든 옵션 체크
- 설치위치는 "C:\dev\python" 또는 "D:\dev\python" 으로 지정하고 "Install" 선택
- 필자는 "C:\nmccm\python" 에 설치를 진행하였다.
- "C:\nmccm\python\projects" 폴더를 만들고 vscode 를 실행한다.
- vscode 메뉴중 File -> Open Folder 클릭 후 위 폴더를 지정 후 확인을 선택
- vscode 메뉴중 Terminal -> New Terminal 선택

### 환경

```bash
OS : Windows 10 Pro (Version : 22H2, OS 빌드 : 19045.3803)
Python : 3.12.1
```

### 정상 설치 여부 체크

```python
print('hello world')
```

코드를 작성하고 first.py 저장, 하단의 Terminal 에서 first.py 파일을 실행하면 아래와 같은 화면이 볼 수 있다.

```python
PS C:\nmccm\python\projects> py first.py
first
```

크롤링을 하기 위해서는 아래의 패키지와 크롬드라이버가 필요하다. 
우선 필요한 패키지를 확인하기 위해 위에서 작성한 print('hello world') 코드를 삭제하고, 아래의 코드를 first.py 작성하고 실행해보자 

```bash
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
import time
from datetime import datetime
from bs4 import BeautifulSoup as bs
from urllib import parse
```

```bash
PS C:\nmccm\python\projects> py first.py
Traceback (most recent call last):
  File "C:\nmccm\python\projects\first.py", line 1, in <module>
    from selenium import webdriver
ModuleNotFoundError: No module named 'selenium'
```

위와 같이 친절하게 selenium 모듈이 없다는 메세지가 출력된다. 아래의 가이드에 따라 하나씩 설치 진행을 해보자. 

```bash
PS C:\nmccm\python\projects> C:\nmccm\python\Scripts\pip.exe install selenium
Collecting selenium
  Obtaining dependency information for selenium from https://files.pythonhosted.org/packages/97/e3/fd7272d6d2c49fd49a79a603cb28c8b5a71f8911861b4a0409b3c006a241/selenium-4.17.2-py3-none-any.whl.metadata
  Downloading selenium-4.17.2-py3-none-any.whl.metadata (6.9 kB)
Collecting urllib3[socks]<3,>=1.26 (from selenium)
  Obtaining dependency information for urllib3[socks]<3,>=1.26 from https://files.pythonhosted.org/packages/88/75/311454fd3317aefe18415f04568edc20218453b709c63c58b9292c71be17/urllib3-2.2.0-py3-none-any.whl.metadata
  Downloading urllib3-2.2.0-py3-none-any.whl.metadata (6.4 kB)
Collecting trio~=0.17 (from selenium)
  Obtaining dependency information for trio~=0.17 from https://files.pythonhosted.org/packages/14/fb/9299cf74953f473a15accfdbe2c15218e766bae8c796f2567c83bae03e98/trio-0.24.0-py3-none-any.whl.metadata
  Downloading trio-0.24.0-py3-none-any.whl.metadata (4.9 kB)
Collecting trio-websocket~=0.9 (from selenium)
  Obtaining dependency information for trio-websocket~=0.9 from https://files.pythonhosted.org/packages/48/be/a9ae5f50cad5b6f85bd2574c2c923730098530096e170c1ce7452394d7aa/trio_websocket-0.11.1-py3-none-any.whl.metadata
  Downloading trio_websocket-0.11.1-py3-none-any.whl.metadata (4.7 kB)
Collecting certifi>=2021.10.8 (from selenium)
  Obtaining dependency information for certifi>=2021.10.8 from https://files.pythonhosted.org/packages/64/62/428ef076be88fa93716b576e4a01f919d25968913e817077a386fcbe4f42/certifi-2023.11.17-py3-none-any.whl.metadata
  Downloading certifi-2023.11.17-py3-none-any.whl.metadata (2.2 kB)
Collecting typing_extensions>=4.9.0 (from selenium)
  Obtaining dependency information for typing_extensions>=4.9.0 from https://files.pythonhosted.org/packages/b7/f4/6a90020cd2d93349b442bfcb657d0dc91eee65491600b2cb1d388bc98e6b/typing_extensions-4.9.0-py3-none-any.whl.metadata  Downloading typing_extensions-4.9.0-py3-none-any.whl.metadata (3.0 kB)
Collecting attrs>=20.1.0 (from trio~=0.17->selenium)
  Obtaining dependency information for attrs>=20.1.0 from https://files.pythonhosted.org/packages/e0/44/827b2a91a5816512fcaf3cc4ebc465ccd5d598c45cefa6703fcf4a79018f/attrs-23.2.0-py3-none-any.whl.metadata
  Downloading attrs-23.2.0-py3-none-any.whl.metadata (9.5 kB)
Collecting sortedcontainers (from trio~=0.17->selenium)
  Downloading sortedcontainers-2.4.0-py2.py3-none-any.whl (29 kB)
Collecting idna (from trio~=0.17->selenium)
  Obtaining dependency information for idna from https://files.pythonhosted.org/packages/c2/e7/a82b05cf63a603df6e68d59ae6a68bf5064484a0718ea5033660af4b54a9/idna-3.6-py3-none-any.whl.metadata
  Downloading idna-3.6-py3-none-any.whl.metadata (9.9 kB)
Collecting outcome (from trio~=0.17->selenium)
  Obtaining dependency information for outcome from https://files.pythonhosted.org/packages/55/8b/5ab7257531a5d830fc8000c476e63c935488d74609b50f9384a643ec0a62/outcome-1.3.0.post0-py2.py3-none-any.whl.metadata
  Downloading outcome-1.3.0.post0-py2.py3-none-any.whl.metadata (2.6 kB)
Collecting sniffio>=1.3.0 (from trio~=0.17->selenium)
  Downloading sniffio-1.3.0-py3-none-any.whl (10 kB)
Collecting cffi>=1.14 (from trio~=0.17->selenium)
  Obtaining dependency information for cffi>=1.14 from https://files.pythonhosted.org/packages/e9/63/e285470a4880a4f36edabe4810057bd4b562c6ddcc165eacf9c3c7210b40/cffi-1.16.0-cp312-cp312-win_amd64.whl.metadata
  Downloading cffi-1.16.0-cp312-cp312-win_amd64.whl.metadata (1.5 kB)
Collecting wsproto>=0.14 (from trio-websocket~=0.9->selenium)
  Downloading wsproto-1.2.0-py3-none-any.whl (24 kB)
Collecting pysocks!=1.5.7,<2.0,>=1.5.6 (from urllib3[socks]<3,>=1.26->selenium)
  Downloading PySocks-1.7.1-py3-none-any.whl (16 kB)
Collecting pycparser (from cffi>=1.14->trio~=0.17->selenium)
  Downloading pycparser-2.21-py2.py3-none-any.whl (118 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 118.7/118.7 kB 7.2 MB/s eta 0:00:00
Collecting h11<1,>=0.9.0 (from wsproto>=0.14->trio-websocket~=0.9->selenium)
  Downloading h11-0.14.0-py3-none-any.whl (58 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 58.3/58.3 kB 3.0 MB/s eta 0:00:00
Downloading selenium-4.17.2-py3-none-any.whl (9.9 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 9.9/9.9 MB 9.7 MB/s eta 0:00:00
Downloading certifi-2023.11.17-py3-none-any.whl (162 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 162.5/162.5 kB 10.2 MB/s eta 0:00:00
Downloading trio-0.24.0-py3-none-any.whl (460 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 460.2/460.2 kB 5.8 MB/s eta 0:00:00
Downloading trio_websocket-0.11.1-py3-none-any.whl (17 kB)
Downloading typing_extensions-4.9.0-py3-none-any.whl (32 kB)
Downloading attrs-23.2.0-py3-none-any.whl (60 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 60.8/60.8 kB 3.4 MB/s eta 0:00:00
Downloading cffi-1.16.0-cp312-cp312-win_amd64.whl (181 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 182.0/182.0 kB 10.7 MB/s eta 0:00:00
Downloading idna-3.6-py3-none-any.whl (61 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 61.6/61.6 kB 3.2 MB/s eta 0:00:00
Downloading outcome-1.3.0.post0-py2.py3-none-any.whl (10 kB)
Downloading urllib3-2.2.0-py3-none-any.whl (120 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 120.9/120.9 kB ? eta 0:00:00
Installing collected packages: sortedcontainers, urllib3, typing_extensions, sniffio, pysocks, pycparser, idna, h11, certifi, attrs, wsproto, outcome, cffi, trio, trio-websocket, selenium
Successfully installed attrs-23.2.0 certifi-2023.11.17 cffi-1.16.0 h11-0.14.0 idna-3.6 outcome-1.3.0.post0 pycparser-2.21 pysocks-1.7.1 selenium-4.17.2 sniffio-1.3.0 sortedcontainers-2.4.0 trio-0.24.0 trio-websocket-0.11.1 typing_extensions-4.9.0 urllib3-2.2.0 wsproto-1.2.0

[notice] A new release of pip is available: 23.2.1 -> 23.3.2
[notice] To update, run: C:\nmccm\python\python.exe -m pip install --upgrade pip
```

첫번째 패키지 설치를 진행하니 하단에 pip 업그레이드가 필요하다는 메세지가 출력된다. pip 업그레이드도 진행한다.

```bash
PS C:\nmccm\python\projects> C:\nmccm\python\python.exe -m pip install --upgrade pip
Requirement already satisfied: pip in c:\nmccm\python\lib\site-packages (23.2.1)
Collecting pip
  Obtaining dependency information for pip from https://files.pythonhosted.org/packages/15/aa/3f4c7bcee2057a76562a5b33ecbd199be08cdb4443a02e26bd2c3cf6fc39/pip-23.3.2-py3-none-any.whl.metadata
  Downloading pip-23.3.2-py3-none-any.whl.metadata (3.5 kB)
Downloading pip-23.3.2-py3-none-any.whl (2.1 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.1/2.1 MB 7.5 MB/s eta 0:00:00
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 23.2.1
    Uninstalling pip-23.2.1:
      Successfully uninstalled pip-23.2.1
  WARNING: The scripts pip.exe, pip3.12.exe and pip3.exe are installed in 'C:\nmccm\python\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed pip-23.3.2
```

```bash
PS C:\nmccm\python\projects> py first.py
Traceback (most recent call last):
  File "C:\nmccm\python\projects\first.py", line 7, in <module>
    from bs4 import BeautifulSoup as bs
ModuleNotFoundError: No module named 'bs4'
```

bs4 module 설치

```bash
PS C:\nmccm\python\projects> C:\nmccm\python\Scripts\pip.exe install BeautifulSoup4 
Collecting BeautifulSoup4
  Downloading beautifulsoup4-4.12.3-py3-none-any.whl.metadata (3.8 kB)
Collecting soupsieve>1.2 (from BeautifulSoup4)
  Downloading soupsieve-2.5-py3-none-any.whl.metadata (4.7 kB)
Downloading beautifulsoup4-4.12.3-py3-none-any.whl (147 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 147.9/147.9 kB 1.8 MB/s eta 0:00:00
Downloading soupsieve-2.5-py3-none-any.whl (36 kB)
Installing collected packages: soupsieve, BeautifulSoup4
Successfully installed BeautifulSoup4-4.12.3 soupsieve-2.5
```

```bash
PS C:\nmccm\python\projects> py first.py
PS C:\nmccm\python\projects> 
```

더이상 패키지 설치 에러가 발생하지 않는다. 이제 크롬드라이버를 다운받아 C:\nmccm\python\projects 폴더에 넣어주자. 크롬드라이버는 https://chromedriver.chromium.org/downloads 에서 다운로드 가능한데, 자신이 사용하는 크롬 브라우저와 버전을 맞춰줘야 한다. 현재 필자가 사용중인 크롬 브라우저는 121.0.6167.86 이고, https://chromedriver.chromium.org/downloads 사이트에서 위 버전의 드라이버를 다운로드 하려면 아래 문구의 "the Chrome for Testing availability dashboard." 링크를 클릭해서 Stable 버전을 다운로드 받아야 한다. (chromedriver.exe)


> Current Releases
If you are using Chrome version 115 or newer, please consult *<u>the Chrome for Testing availability dashboard.</u>* This page provides convenient JSON endpoints for specific ChromeDriver version downloading. For older versions of Chrome, please see below for the version of ChromeDriver that supports it. For more information on selecting the right version of ChromeDriver, please see the Version Selection page.


이제 본격적으로 크롤링을 해보자. 크롤링 대상은 github 사이트의 현재 원화 기준의 유로 환율 제공 페이지 (https://github.com/fawazahmed0/currency-api/blob/1/2024-01-31/currencies/eur/krw.json) 이고, 긁어올 내용은 아래와 같다. 

```bash
{
    "date": "2024-01-31",
    "krw": 1443.73399355
}
```

위 내용을 선택하여 크롬 개발자 도구(F12) 확인해보면 코드가 다음과 같다. 

```html
<div id="copilot-button-positioner" class="Box-sc-g0xbh4-0 cluMzC">
	<div class="Box-sc-g0xbh4-0 eRkHwF">
		<textarea id="read-only-cursor-text-area" aria-label="file content" aria-readonly="true" inputmode="none" tabindex="0" aria-multiline="true" aria-haspopup="false" data-gramm="false" data-gramm_editor="false" data-enable-grammarly="false" spellcheck="false" autocorrect="off" autocapitalize="off" autocomplete="off" data-ms-editor="false" class="react-blob-textarea react-blob-print-hide" style="resize: none; margin-top: -2px; padding-left: 92px; padding-right: 70px; width: 100%; background-color: unset; box-sizing: border-box; color: transparent; position: absolute; border: none; tab-size: 8; outline: none; overflow: auto hidden; height: 100px; font-size: 12px; line-height: 20px; overflow-wrap: normal; overscroll-behavior-x: none; white-space: pre; z-index: 1;">{ "date": "2024-01-31", "krw": 1443.73399355 }</textarea>
	</div>
</div>
```

html 은 제거된 텍스트만 추출할것이기 때문에 " id='read-only-cursor-text-area' " 부분을 셀렉하면 되겠다.

```python
PS C:\nmccm\python\projects> py first.py

DevTools listening on ws://127.0.0.1:52544/devtools/browser/c9fb630e-25a7-40a1-8465-87abc14cc62e
target url : https://github.com/fawazahmed0/currency-api/blob/1/2024-01-31/currencies/eur/krw.json
====== content start ======
{
    "date": "2024-01-31",
    "krw": 1443.73399355
}
====== content end ======
```

정상적으로 잘 되는것 같다. 최종소스는 아래에....

```bash
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
import time
from datetime import datetime
from bs4 import BeautifulSoup as bs
from urllib import parse

driver = webdriver.Chrome()
url = "https://github.com/fawazahmed0/currency-api/blob/1/2024-01-31/currencies/eur/krw.json"
print('target url : ' + url)
driver.get(url)
time.sleep(3) 
plist = driver.find_elements(By.ID, "read-only-cursor-text-area") 

for i in plist:        
    text = str(i.get_attribute('innerHTML'))
    print('====== content start ======')
    print(text)
    print('====== content end ======')
```

