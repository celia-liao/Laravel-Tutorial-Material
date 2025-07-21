---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
marp: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

# Laravel Day 9  
### 表單錯誤處理與 Flash 訊息

---

## 🎯 今天的任務

✔ 每頁可插入獨立樣式  
✔ 顯示欄位錯誤訊息  
✔ 保留輸入值  
✔ 顯示新增成功通知  
✔ **這一章使用 Route 處理表單邏輯**

---

## 1️⃣ layouts 樣板加上樣式插槽

### 修改檔案：`resources/views/layouts/app.blade.php`

<small>

```php
<!-- layouts/app.blade.php -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Task List APP</title>
    @yield('styles') <!-- 個別頁面的樣式插這裡 -->
</head>
<body>
    <h1>@yield('title')</h1>
    <div>
        @yield('content')
    </div>
</body>
</html>

````


</small>

---

## 2️⃣ 表單驗證 → 修改檔案：`routes/web.php`
<small>

```php
use Illuminate\Http\Request;
use App\Models\Task;

Route::post('/tasks', function (Request $request) {
    // ✅ 驗證
    $data = $request->validate([
        'title' => 'required|max:255',
        'description' => 'required',
        'long_description' => 'required',
    ]);

    // ✅ 儲存
    Task::create($data);

    // ✅ 成功訊息存入 session
    return redirect()->route('tasks.create')
        ->with('success', '任務新增成功！');
})->name('tasks.store');
```

</small>

---

## 3️⃣ 成功訊息（Flash Message）原理

```php
return redirect()->route('tasks.create')
    ->with('success', '任務新增成功');
```

* Laravel 會把 success 訊息放進 session
* 頁面顯示後就會自動清掉（只顯示一次）

---

## 4️⃣ 表單畫面顯示錯誤與成功訊息 → 修改`resources/views/tasks/create.blade.php`

<small>

```php
@extends('layouts.app')

@section('title', 'Add Task')

@section('styles')
<style>
  .error-msg { color: red; font-size: 0.8rem; }
  .success-msg { color: green; font-weight: bold; margin-bottom: 10px; }
</style>
@endsection

@section('content')
  {{-- ✅ 成功訊息 --}}
  @if (session('success'))
    <div class="success-msg">{{ session('success') }}</div>
  @endif
  <form method="POST" action="{{ route('tasks.store') }}">
    @csrf
    <div>
      <label for="title">Title</label>
      <input name="title" value="{{ old('title') }}">
      @error('title') <p class="error-msg">{{ $message }}</p> @enderror
    </div>
    <!-- 你可以再加 description, long_description -->
  </form>
@endsection
```


</small>

---

## 5️⃣ Laravel 機制回顧：它幫你做了什麼？

| 動作         | Laravel 幫你做的事                           |
| ---------- | --------------------------------------- |
| 驗證失敗       | 自動導回上一頁 + 錯誤訊息寫入 session                |
| Blade 顯示錯誤 | `@error('欄位')` 顯示錯誤，`old()` 保留輸入值       |
| 驗證成功       | `->with('success', ...)` 成功訊息寫入 session |
| Blade 顯示成功 | `session('success')` 顯示通知訊息             |

---

## 🎯 小結

| 功能   | Blade 寫法                                  | 說明                  |
| ---- | ----------------------------------------- | ------------------- |
| 顯示錯誤 | `@error('欄位')`                            | 顯示該欄位錯誤訊息           |
| 保留輸入 | `old('欄位')`                               | 顯示剛剛輸入但有錯誤的內容       |
| 成功訊息 | `session('success')` + `->with()`         | 成功訊息（只顯示一次）         |
| 樣式管理 | `@section('styles')` / `@yield('styles')` | 子頁面自帶 CSS，不影響其他頁面樣式 |

