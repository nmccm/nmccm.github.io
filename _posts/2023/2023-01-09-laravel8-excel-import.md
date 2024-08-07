---
title: "Laravel Excel Import"
categories:
- PHP
- Laravel
tags:
- Laravel8
- Excel
---

일반 사용자 화면을 포함하여 많은 사람들에게 사랑받는 엑셀 포맷으로 서버에 대량으로 데이터를 업로드 하는 기능은 이제 필수 기능이 되어 버렸다. 아래 코드는 라라벨을 사용하여 엑셀을 임포트 하는 방법에 대해 기술한다. 즉, 라라벨 프레임워크 8 버전에서 엑셀 업로드 및 읽어들이는 방법에 대해 기술한다.

### 1. 환경

```bash
OS : window 10 pro (with WSL1)
Apache : 2.4
PHP : 7.3.27
MariaDB : 10.4.18
```
### 2. HTML & Javascript & Jqeury

새로운 뷰를 하나 만든다. 컨트롤러 진입없이 바로 라우터에서 뷰 화면을 출력 가능하다.

```html
<head>    
    <link rel="stylesheet" href="/css/plugin/jquery-confirm.min.css">
    <script src="/js/plugin/jquery.min.js"></script>  
    <script src="/js/plugin/jquery-confirm.min.js"></script>
</head>
<form id="upload-excel-file-form" onsubmit="return false;" enctype="multipart/form-data">
    <h2>대량 등록</h2>
    <div>name : <input type="text" name="filename" value=""></div>
    <div>file : <input type="file" id="file1" name="product_multi_register" value=""></div>
    <div>&nbsp;</div>
    <div><button id="upload-excel-file-submit-btn">등록</button></div>
</form>
```

upload-excel-file-submit-btn 을 id 로 지정된 버튼을 클릭하면 Jquery confirm 뜨고, Yes 선택하면 productMultiRegisterProc 네임을 가진 컨트롤러에 요청을 보내는 코드이다.

```javascript
$("#upload-excel-file-submit-btn").click(() => {
    $.confirm({
        title: 'Confirm!',
        content: '등록하시겠습니까?',
        buttons: {
            Yes: () => {
                let formData = new FormData();
                let files = $('#file1')[0].files;
                let eleName = $('#file1')[0].name;
                formData.append(eleName, files[0]);

                if(files.length <= 0) {
                    alert("Please select a file.");
                    return false;
                }

                $.ajax({
                    url: '{{ route('productMultiRegisterProc') }}',
                    type: 'post',
                    data: formData,
                    contentType: false,
                    processData: false,
                    success: (res) => {
                        console.log(res);
                        const msg = res.code ? 'success' : 'error';
                        alert(msg);
                    },
                    beforeSend: showLoading,
                    complete: hideLoading,
                    error: hideLoading,
                });
            },
            No: () => {
                alert('Canceled!');
            }
        }
    });
});
```

### 3. 라라벨 라우터 설정 

http://domain/product/multiRegisterProc 주소에 대한 별칭으로 productMultiRegisterProc 설정하였다. 위 자바스크립트에서는 앞서 언급한 주소로 호출할 것이다.

```php
Route::group(['prefix' => 'product'], function() {       
    Route::post('multiRegisterProc', [ProductController::class, 'multiRegisterProc'])
        ->name('productMultiRegisterProc');       
});
```

### 4. 라라벨 컨트롤러 작성

```php
use App\Imports\ExcelImport;

public function multiRegisterProc(Request $request) {
    try {
        $eleName = 'product_multi_register';
        $isSuccess = false;

        if(!$request->hasFile($eleName)) {
            return response()->json([
                'code' => $isSuccess,
                'msg' => $eleName . ' is null',
                'data' => $request->all(),
            ]);
        }

        $obj = $request->file($eleName);        
        $filename = $obj->store('public/excel/upload'); // storage/app/public/excel/upload
        $file = storage_path('app/') . $filename; // storage/app/
        $rows = Excel::toArray(new ExcelImport, $file);

        if(sizeof($rows) > 0 && sizeof($rows[0]) > 0) {
            foreach($rows[0] as $row) {
                $model = new Product([
                    'name' => $row[0]
                ]);
                $model->save();
            }
        }

        return response()->json([
            'code' => true,
            'msg' => 'success',
            'data' => $rows[0],
        ]);
    }
    catch(\Exception $e) {
        log::error(__METHOD__ . " : " . $e->getMessage());
    }
}
```

### 5. 엑셀 핸들러

ToModel 을 implements 하였으므로, 반드시 model 메쏘드를 재 정의 해줘야 한다.

```php
namespace App\Imports;

use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Maatwebsite\Excel\Concerns\ToModel;

class ExcelImport implements ToModel
{
    public function model(array $row) {
    }
}
```
