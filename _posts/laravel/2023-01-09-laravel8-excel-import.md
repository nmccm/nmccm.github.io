---
title: "Laravel Excel Import"
categories:
- Laravel
tags:
- Laravel8
- Excel
- PHP
---

환경 : Laravel 8.83.26, WSL1 (Apache 2.4, PHP 7.3.27, MariaDB 10.4.18)

라라벨 프레임워크 8 버전에서 엑셀 업로드 및 읽어들이는 방법

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
- web.php
```php
Route::group(['prefix' => 'product'], function() {       
    Route::post('multiRegisterProc', [ProductController::class, 'multiRegisterProc'])
        ->name('productMultiRegisterProc');       
});
```

- ProductController
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
- ExcelImport
```php
namespace App\Imports;

use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Maatwebsite\Excel\Concerns\ToModel;

class ExcelImport implements ToModel
{
    /**
     * @param array $row
     * @return \Illuminate\Database\Eloquent\Model|\Illuminate\Database\Eloquent\Model[]|void|null
     */
    public function model(array $row) {
    }
}
```