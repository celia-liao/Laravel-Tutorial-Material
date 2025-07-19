---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
marp: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

# Laravel Day 8

### 表單送出與資料儲存

---

## 🎯 這次我們要完成什麼？

* 練習用 Blade 表單送資料
* 從 Route 取得輸入資料並驗證
* 存進資料庫並跳轉回詳情頁
* 認識 `@csrf`、Request 驗證、重定向

---

## 1. 建立任務表單頁（create.blade.php）

```blade
@extends('layouts.app')

@section('title', 'Add Task')

@section('content')
  <form method="POST" action="{{ route('tasks.store') }}">
    @csrf

    <div>
      <label for="title">Title</label>
      <input type="text" name="title" id="title" />
    </div>

    <div>
      <label for="description">Description</label>
      <textarea name="description" id="description" rows="5"></textarea>
    </div>

    <div>
      <label for="long_description">Long Description</label>
      <textarea name="long_description" id="long_description" rows="10"></textarea>
    </div>

    <div>
      <button type="submit">Add Task</button>
    </div>
  </form>
@endsection
```

---

## 2. 路由設定：表單顯示與資料儲存

```php
use Illuminate\Http\Request;

// 顯示新增任務表單
Route::get('/tasks/create', function () {
    return view('create');
})->name('tasks.create');

// 處理表單送出
Route::post('/tasks', function (Request $request) {
    $data = $request->validate([
        'title'=> 'required|max:255',
        'description' => 'required',
        'long_description' => 'required',
    ]);

    $task = new Task;
    $task->title = $data['title'];
    $task->description = $data['description'];
    $task->long_description = $data['long_description'];
    $task->save();

    return redirect()->route('tasks.show', ['id' => $task->id]);
})->name('tasks.store');
```

---

## 3. 說明

| 功能                       | 說明           |
| ------------------------ | ------------ |
| `@csrf`                  | 表單防止 [CSRF 攻擊](https://www.explainthis.io/zh-hant/swe/what-is-csrf) |
| `$request->validate()`   | 驗證輸入資料       |
| `new Task`               | 建立任務模型實  |
| `$task->save();`         | 將目前這個 $task 物件的所有屬性（欄位）存進資料庫 |
| `redirect()->route(...)` | 導回任務詳情頁      |

---

## 補充 | 路由匹配規則與排序

<small>

### 路由排序原則

* **具體路由（靜態網址）優先**：如 `/tasks/create`、`/tasks/edit`
* **參數路由（動態網址）在後**：如 `/tasks/{id}`、`/users/{id}`

### 為什麼要這樣排序？

* 如果參數路由（`/tasks/{id}`）**寫在前面**，像 `/tasks/create` 這種靜態網址會被當作參數 `{id}` 匹配。

  * 會嘗試找 id="create" 的任務，找不到就回 404!

### 正確做法： 先寫 `/tasks/create` ; 再寫 `/tasks/{id}`

</small>  

---

## 4️. 表單驗證規則範例

```php
$request->validate([
    'title' => 'required|max:255',
    'description' => 'required',
    'long_description' => 'required',
]);
```
- required：這個欄位必須要有內容（不可為空）
- max:255：這個欄位的字數不能超過 255 字
- 多個規則以 | 符號連接（例如 required|max:255）

---

## 🎯 小結

| 步驟  | 說明                              |
| --- | ------------------------------- |
| 1️⃣ | 表單畫面建立（create.blade.php）        |
| 2️⃣ | 建立 `GET` 路由 `/tasks/create`     |
| 3️⃣ | 建立 `POST` 路由 `/tasks`，處理資料驗證與儲存 |
| 4️⃣ | 儲存成功後導向任務詳情頁（show）              |

