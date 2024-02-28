---
title: "파이썬 엑셀파일 내용 단순 비교 프로그램(Python)"
categories:
- Python
tags:
- Python
- Excel
---

### 1. 환경

```bash
OS : window 11 pro
OS 빌드 : 22621.3007
Python : 3.12.1
Pyinstaller : 6.3.0
```
### 2. 개발배경

유관부서에서 웹 기반 판매 관리 프로그램 개발요청이 옴. 판매관리 프로그램이므로 재고 수불에 대한 내역도 별도의 화면에서 출력되어야 하는데, 이 재고 파일의 양이 엑셀로 6개의 시트이고, 각 시트당 35,000건 정도의 데이터가 어마어마하게 저장되어 있다. 요구사항중 첫번째로 별도의 수정없이 엑셀파일을 업로드 하면 파싱하여 데이터베이스에 저장하는것과 두번째로 그 모든 파일을 수시로 다운로드 받아야 된다는점이 너무나 마음에 걸렸지만, 일단 단순무식한 방법으로 통 파일을 웹에 전송하는 기능을 개발하였고(실제 개발은 팀원이... -_-a), 잘 작동은 하였지만 서버 모니터링 결과 서버 리소스 사용량이 적지 않았다. 저 무식한 파일을 매일 업/다운로드 하지 않는 방법을 생각해야했고, 업/다운로드에 대한 이유를 들어보니 단순 재고 변동량을 수기로 하기에 벅차다는 이유였다. (엑셀 VLoopup 이나 XLoopup 으로 될것 같은데, 현업에서 안된다고 하니...) 하여 전일재고파일과 금일재고파일을 비교하여 변경사항만 별도의 엑셀파일로 저장하는 프로그램을 개발하기로 했다.

### 3. 프로젝트 폴더 생성 및 이동

```bash
PS C:\nmccm\python\projects> mkdir diff_file
PS C:\nmccm\python\projects> cd diff_file
```

### 4. PYINSTALLER 설치

```bash
PS C:\nmccm\python\projects\diff_file> pip install pyinstaller
Collecting pyinstaller
  Downloading pyinstaller-6.3.0-py3-none-win_amd64.whl.metadata (8.3 kB)
Collecting setuptools>=42.0.0 (from pyinstaller)
  Downloading setuptools-69.0.3-py3-none-any.whl.metadata (6.3 kB)
Collecting altgraph (from pyinstaller)
  Downloading altgraph-0.17.4-py2.py3-none-any.whl.metadata (7.3 kB)
Collecting pyinstaller-hooks-contrib>=2021.4 (from pyinstaller)
  Downloading pyinstaller_hooks_contrib-2024.0-py2.py3-none-any.whl.metadata (16 kB)
Collecting packaging>=22.0 (from pyinstaller)
  Downloading packaging-23.2-py3-none-any.whl.metadata (3.2 kB)
Collecting pefile>=2022.5.30 (from pyinstaller)
  Downloading pefile-2023.2.7-py3-none-any.whl (71 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 71.8/71.8 kB 1.9 MB/s eta 0:00:00
Collecting pywin32-ctypes>=0.2.1 (from pyinstaller)
  Downloading pywin32_ctypes-0.2.2-py3-none-any.whl.metadata (3.8 kB)
Downloading pyinstaller-6.3.0-py3-none-win_amd64.whl (1.3 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.3/1.3 MB 4.1 MB/s eta 0:00:00
Downloading packaging-23.2-py3-none-any.whl (53 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 53.0/53.0 kB ? eta 0:00:00
Downloading pyinstaller_hooks_contrib-2024.0-py2.py3-none-any.whl (326 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 326.8/326.8 kB 5.0 MB/s eta 0:00:00
Downloading pywin32_ctypes-0.2.2-py3-none-any.whl (30 kB)
Downloading setuptools-69.0.3-py3-none-any.whl (819 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 819.5/819.5 kB 6.5 MB/s eta 0:00:00
Downloading altgraph-0.17.4-py2.py3-none-any.whl (21 kB)
Installing collected packages: altgraph, setuptools, pywin32-ctypes, pefile, packaging, pyinstaller-hooks-contrib, pyinstaller
Successfully installed altgraph-0.17.4 packaging-23.2 pefile-2023.2.7 pyinstaller-6.3.0 pyinstaller-hooks-contrib-2024.0 pywin32-ctypes-0.2.2 setuptools-69.0.3
```

### 5. 소스

아래의 소스를 file_diff.py 파일명으로 저장

```python
# Author : nmccm (nmccm@naver.com)
# Date : 2024-02-01
# 
# 두개의 엑셀파일을 비교하여 이격 데이터만 별도 파일로 저장하는 프로그램 (비교컬럼 : A=상품코드, C=사방넷코드(6), Q=당일재고)
#
# 조건 1 : 두개의 파일 중 첫번째 파일 기준으로 당일 재고가 변경된 데이터를 찾아 파일에 저장
# 조건 2 : 두개의 파일 중 첫번째 파일 기준으로 신규 데이터를 찾아 파일에 저장
# 조건 3 : 두개의 파일 중 첫번째 파일 기준으로 사방넷 코드가 변경된 데이터를 찾아 파일에 저장

import sys
import os.path
import pandas as pd
from openpyxl import Workbook
from datetime import datetime

def _print(str, newLine = False):
    print(str, sep = '', end = ',', file = sys.stdout, flush = True)

def printLine():
    print('-----------------------------------------------------------------------------------------------------------------')

def printLogTitle():
    print('Flag\t\t\tLine\t\tCode\t\tQty\t\tCurrent\t\tSBN\t\tCurrent')

def printResultTitle():
    print('Flag\t\t\tTotal Count')

def printResult(flag1 = 0, flag2 = 0, flag3 = 0):
    print('Inconsistency\t\t' + str(flag1))
    print('Sabangnet\t\t' + str(flag2))
    print('New Item\t\t' + str(flag3))

_def = ""
now = datetime.now()

if len(sys.argv) < 3:
    print('ERROR : 올바른 형식으로 다시 시도해주세요.')
    print('ex> file_diff.exe 전일재고파일명.xls 금일재고파일명.xls')
else:    
    if os.path.isfile('./' + str(sys.argv[1])):

        if os.path.isfile('./' + str(sys.argv[2])):                        

            workbook = Workbook()
            sheet = workbook.active
            sheet['A1'] = "상품코드"
            sheet['B1'] = "사진코드"
            sheet['C1'] = "사방넷코드(6)"
            sheet['D1'] = "사방넷코드(10)"
            sheet['E1'] = "구분"
            sheet['F1'] = "브랜드(영문)"
            sheet['G1'] = "브랜드(한글)"
            sheet['H1'] = "카테고리(영문)"
            sheet['I1'] = "카테고리(한글)"
            sheet['J1'] = "시즌"
            sheet['K1'] = "사이즈"
            sheet['L1'] = "상품명"
            sheet['M1'] = "차수제거"
            sheet['N1'] = "원가"
            sheet['O1'] = "전일재고"
            sheet['P1'] = "전일금액"
            sheet['Q1'] = "당일재고"
            sheet['R1'] = "당일금액"                        
            sheet['S1'] = "Result"   
            df1 = pd.read_excel('./' + str(sys.argv[1]), sheet_name = 0)
            df2 = pd.read_excel('./' + str(sys.argv[2]), sheet_name = 0)
            index = 0
            resultInconsistencyCount = 0
            resultSabangnetCount = 0
            resultNewItemCount = 0

            print(_def)
            printLogTitle()
            printLine()
     
            for item2 in df2.values.tolist():                    
                index = index + 1                
                isMatch = 0
                item1Qty = 0
                item2Qty = item2[16]               

                for item1 in df1.values.tolist():                                    
                    item1Qty = item1[16]                                        

                    if str(item1[0]) == str(item2[0]):
                        isMatch = 1
                        if item1[16] != item2[16]:
                            item2Result = item2 + [str(item1[16])]                            
                            resultInconsistencyCount = resultInconsistencyCount + 1
                            sheet.append(item2Result)                            
                            print('Inconsistency\t\t' + str(index) + '\t\t' + str(item2[0]) + '\t\t' + str(item1Qty) + '\t\t' + str(item2Qty))
                        elif item1[16] == item2[16] and len(str(item2[2])) > 6 and item1[2] != item2[2]:
                            item2Result = item2 + ['SBN']
                            resultSabangnetCount = resultSabangnetCount + 1
                            sheet.append(item2Result)                                         
                            print('Sabangnet\t\t' + str(index) + '\t\t' + str(item2[0]) + '\t\t' + str(item1Qty) + '\t\t' + str(item2Qty) + '\t\t' + str(item1[2]) + '\t\t' + str(item2[2]))

                if isMatch < 1:                  
                    resultNewItemCount = resultNewItemCount + 1  
                    sheet.append(item2 + ['NEW'])
                    print('New Item\t\t' + str(index) + '\t\t' + str(item2[0]) + '\t\t' + str(item1Qty) + '\t\t' + str(item2Qty))
                else:
                    isMatch = 0

            workbook.save("result-" + str(now.timestamp()) + ".xlsx")            
            print(_def)
            print(_def)
            printResultTitle()
            printLine()
            printResult(resultInconsistencyCount, resultSabangnetCount, resultNewItemCount)
            print(_def)

        else:
            print('ERROR : 금일재고파일(' + str(sys.argv[2]) + ')을 찾을 수 없습니다.')      
            
    else:
        print('ERROR : 전일재고파일(' + str(sys.argv[1]) + ')을 찾을 수 없습니다.')                

    
# end        
```

### 6. 테스트

```bash
PS C:\nmccm\python\projects\diff_file> py file_diff.py
ERROR : 올바른 형식으로 다시 시도해주세요.
ex> file_diff.exe 전일재고파일명.xls 금일재고파일명.xls

PS C:\nmccm\python\projects\diff_file> py file_diff.py 11 22
ERROR : 전일재고파일(11)을 찾을 수 없습니다.

PS C:\nmccm\python\projects\diff_file> py file_diff.py 1.xlsm 22
ERROR : 금일재고파일(22)을 찾을 수 없습니다.

PS C:\nmccm\python\projects\diff_file> py file_diff.py 1.xlsm 2.xlsm

Flag                    Line            Code            Qty             Current         SBN             Current
-----------------------------------------------------------------------------------------------------------------
Inconsistency           2               121256          7               77
New Item                5               128553          0               0


Flag                    Total Count
-----------------------------------------------------------------------------------------------------------------
Inconsistency           1
Sabangnet               0
New Item                1
```

### 7. 빌드

```bash
PS C:\nmccm\python\projects\diff_file> pyinstaller.exe file_diff.py
403 INFO: PyInstaller: 6.3.0
403 INFO: Python: 3.12.1
450 INFO: Platform: Windows-11-10.0.22621-SP0
451 INFO: wrote C:\nmccm\python\projects\diff_file\file_diff.spec
880 INFO: checking Analysis
881 INFO: Building Analysis because Analysis-00.toc is non existent
881 INFO: Initializing module dependency graph...
882 INFO: Caching module graph hooks...
911 INFO: Analyzing base_library.zip ...
.
.
.
5822 INFO: Caching module dependency graph...
5914 INFO: Running Analysis Analysis-00.toc
5915 INFO: Looking for Python shared library...
5921 INFO: Using Python shared library: C:\nmccm\python\python312.dll
5921 INFO: Analyzing C:\nmccm\python\projects\diff_file\file_diff.py
.
.
.
35299 INFO: checking PYZ
35300 INFO: Building PYZ because PYZ-00.toc is non existent
36481 INFO: checking PKG
36481 INFO: Building PKG because PKG-00.toc is non existent
36481 INFO: Building PKG (CArchive) file_diff.pkg
36514 INFO: Building PKG (CArchive) file_diff.pkg completed successfully.
36515 INFO: checking EXE
36515 INFO: Building EXE because EXE-00.toc is non existent
36515 INFO: Building EXE from EXE-00.toc
36643 INFO: Copying icon to EXE
36707 INFO: Copying 0 resources to EXE
36707 INFO: Embedding manifest in EXE
36770 INFO: Appending PKG archive to EXE
37063 INFO: Fixing EXE headers
37392 INFO: Building EXE from EXE-00.toc completed successfully.
37402 INFO: checking COLLECT
37403 INFO: Building COLLECT because COLLECT-00.toc is non existent
37403 INFO: Building COLLECT COLLECT-00.toc
39815 INFO: Building COLLECT COLLECT-00.toc completed successfully.
```

### 8. 배포

정상적으로 빌드가 완료되면 빌드실행한 폴더(C:\nmccm\python\projects\diff_file)에 build, dist, file_diff.spec 파일 및 폴더가 생성된것을 확인할 수 있다. 

```bash
PS C:\nmccm\python\projects\diff_file> dir


    디렉터리: C:\nmccm\python\projects\diff_file


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----      2024-02-02   오전 9:17                build
d-----      2024-02-02   오전 9:18                dist
-a----      2024-02-01   오후 4:28        1673810 1.xlsm
-a----      2024-02-01  오전 11:53        1673877 2.xlsm
-a----      2024-02-01   오후 4:40        4872160 3.xlsm
-a----      2024-02-01   오후 2:50        4872087 4.xlsm
-a----      2024-02-01   오후 5:10           5405 file_diff.py
-a----      2024-02-02   오전 9:17            775 file_diff.spec
```

이제 사용자에게 아래의 파일을 배포하면 된다. 주의할점은 동일 폴더내에 있는 _internal 폴더도 같이 전달해야하고, 실행시에도 반드시 _internal 폴더가 있는 위치에서 실행해야 한다는 점을 알려주자

```bash
PS C:\nmccm\python\projects\diff_file\dist\file_diff> dir


    디렉터리: C:\nmccm\python\projects\diff_file\dist\file_diff


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----      2024-02-02   오전 9:18                _internal
-a----      2024-02-02   오전 9:18       10294134 file_diff.exe
```

### 9. 문제점

아무래도 40,000건에 가까운 로우수를 읽어들이고, 또 건당 40,000건과 비교를 하다보니 Cache 및 Buffer 가 한계에 다다르면 느려지는 현상이 발생 (위 건수 기준 15분에서 최대 1시간까지도 걸린다 -_-)


### 10. 개선

코드를 조금 손봤더니 1분 내로 처리되었다. 아무래도 파이썬에 익숙해지지 않아서 생긴 문제로 추측된다.

```bash
PS C:\nmccm\python\projects\diff_file> vi file_diff_def.py

def getInconsistencyString() : 
    return 'Inconsistency'

def getSabangnetString() : 
    return 'Sabangnet'

def getNewItemString() : 
    return 'New Item'

def getFilenamePrefix() :
    return 'result-'

def getFileExt() : 
    return 'xlsx'

def printLogTitle() :
    print('Flag\t\t\tLine\t\tCode\t\tQty\t\tCurrent\t\tSBN\t\tCurrent')

def printResultTitle() :
    print('Flag\t\t\tTotal Count')

def printResult(flag1 = 0, flag2 = 0, flag3 = 0) :
    print(getInconsistencyString() + '\t\t' + str(flag1))
    print(getSabangnetString() + '\t\t' + str(flag2))
    print(getNewItemString() + '\t\t' + str(flag3))

def printLine() : 
    print('-----------------------------------------------------------------------------------------------------------------')

def getErrorMsg(code = 0, arg = '') :
    if code == 1001 :
        return 'ERROR : 금일재고파일(' + str(arg) + ')을 찾을 수 없습니다.'    
    if code == 1002 :
        return 'ERROR : 전일재고파일(' + str(arg) + ')을 찾을 수 없습니다.'
    if code == 1003 : 
        return 'ERROR : 올바른 형식으로 다시 시도해주세요.'
    
def getExampleString() :
    return 'ex> file_diff.exe 전일재고파일명.xls 금일재고파일명.xls'

def createSheetTitle(sheet) :
    sheet['A1'] = "상품코드"
    sheet['B1'] = "사진코드"
    sheet['C1'] = "사방넷코드(6)"
    sheet['D1'] = "사방넷코드(10)"
    sheet['E1'] = "구분"
    sheet['F1'] = "브랜드(영문)"
    sheet['G1'] = "브랜드(한글)"
    sheet['H1'] = "카테고리(영문)"
    sheet['I1'] = "카테고리(한글)"
    sheet['J1'] = "시즌"
    sheet['K1'] = "사이즈"
    sheet['L1'] = "상품명"
    sheet['M1'] = "차수제거"
    sheet['N1'] = "원가"
    sheet['O1'] = "전일재고"
    sheet['P1'] = "전일금액"
    sheet['Q1'] = "당일재고"
    sheet['R1'] = "당일금액"                        
    sheet['S1'] = "Result"       
    return sheet

def get_def() :
    return ''

def get_dot() :
    return '.'

def get_slash() :
    return '/'

def get_new() :
    return 'NEW'

def get_sbn() :
    return 'SBN'
```

```bash
PS C:\nmccm\python\projects\diff_file> vi file_diff.py

# Author : nmccm (nmccm@naver.com)
# Date : 2024-02-01
# 
# 두개의 엑셀파일을 비교하여 이격 데이터만 별도 파일로 저장하는 프로그램 (비교컬럼 : A=상품코드, C=사방넷코드(6), Q=당일재고)
#
# 조건 1 : 두개의 파일 중 첫번째 파일 기준으로 당일 재고가 변경된 데이터를 찾아 파일에 저장
# 조건 2 : 두개의 파일 중 첫번째 파일 기준으로 신규 데이터를 찾아 파일에 저장
# 조건 3 : 두개의 파일 중 첫번째 파일 기준으로 사방넷 코드가 변경된 데이터를 찾아 파일에 저장

import sys
import os.path
import pandas as pd
from openpyxl import Workbook
from datetime import datetime
from file_diff_def import *
import numpy as np
_def = get_def()
_dot = get_dot()
_slash = get_slash()
_new = get_new()
_sbn = get_sbn()

now = datetime.now()

if len(sys.argv) < 3:
    print(getErrorMsg(1003))
    print(getExampleString())
else:    
    if os.path.isfile(_dot + _slash + str(sys.argv[1])):

        if os.path.isfile(_dot + _slash + str(sys.argv[2])):                        

            workbook = Workbook()
            sheet = workbook.active
            sheet = createSheetTitle(sheet)
            df1 = pd.read_excel(_dot + _slash + str(sys.argv[1]), sheet_name = 0)
            df2 = pd.read_excel(_dot + _slash + str(sys.argv[2]), sheet_name = 0)
            df1List = df1.values.tolist()
            df2List = df2.values.tolist()
            index = 0
            resultInconsistencyCount = 0
            resultSabangnetCount = 0
            resultNewItemCount = 0

            print(_def)
            printLogTitle()
            printLine()

            oldNetForceCodeList = []
            oldDic = {}
            
            # 빠르게 찾기 위하여 전일재고의 넷포스 코드 배열 재 생성 및 딕셔너리 생성
            for item in df1List : 
                oldNetForceCodeList.append(item[0])
                oldDic[item[0]] = item
     
            for item2 in df2List :
                index = index + 1 

                if item2[0] in oldNetForceCodeList :                                     
                    tmpItem1 = oldDic.get(item2[0])

                    if np.isnan(item2[2]) == False : 
                        newSabangnetCode = str(item2[2])[0:6]
                    else:
                        newSabangnetCode = get_def()

                    if np.isnan(tmpItem1[2]) == False :                     
                        oldSabangnetCode = str(tmpItem1[2])[0:6]
                    else:
                        oldSabangnetCode = get_def()                        

                    if tmpItem1[16] != item2[16] :                        
                        resultInconsistencyCount = resultInconsistencyCount + 1                                        
                        sheet.append(item2 + [str(tmpItem1[16])])        
                        print(getInconsistencyString() + '\t\t' + str(index) + '\t\t' + str(item2[0]) + '\t\t' + str(tmpItem1[16]) + '\t\t' + str(item2[16]) + '\t\t' + str(oldSabangnetCode) + '\t\t' + str(newSabangnetCode))                    
                    else :                                                  
                        if str(oldSabangnetCode) != str(newSabangnetCode) : 
                            resultSabangnetCount = resultSabangnetCount + 1
                            sheet.append(item2 + [_sbn])
                            print(getSabangnetString() + '\t\t' + str(index) + '\t\t' + str(item2[0]) + '\t\t' + str(tmpItem1[16]) + '\t\t' + str(item2[16]) + '\t\t' + str(oldSabangnetCode) + '\t\t' + str(newSabangnetCode))                            

                else :
                    resultNewItemCount = resultNewItemCount + 1  
                    sheet.append(item2 + [_new])
                    print(getNewItemString() + '\t\t' + str(index) + '\t\t' + str(item2[0]) + '\t\t' + str(0) + '\t\t' + str(item2[16]) + '\t\t' + str('') + '\t\t' + str(''))

            workbook.save(getFilenamePrefix() + str(now.timestamp()) + "." + getFileExt())            
            print(_def)
            print(_def)
            printResultTitle()
            printLine()
            printResult(resultInconsistencyCount, resultSabangnetCount, resultNewItemCount)
            print(_def)

        else:
            print(getErrorMsg(1001, str(sys.argv[2])))            
            
    else:
        print(getErrorMsg(1002, str(sys.argv[1])))            
    
# end        
```