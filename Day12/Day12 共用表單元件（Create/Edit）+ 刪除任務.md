---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
marp: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')



---
# 共用表單元件(Create/Edit)+ 刪除任務

---

## 🎯 為什麼要這樣做？

- 避免重複撰寫「新增 / 編輯任務」表單  
- 使用同一份 Blade 模板 + 條件判斷，讓表單自動切換狀態  
- 編輯畫面加上「刪除」功能，讓操作流程更完整  

---

## 共用模板檔案：`form.blade.php`

這個表單模板會依據 `$task` 是否存在，自動決定：

- 是否顯示預設值（編輯模式）
- 使用 `POST` 還是 `PUT`
- 是否顯示「刪除」按鈕  

---

## 表單上半部（標題 + 錯誤處理）

```blade
@extends('layouts.app')

@section('title', isset($task) ? 'Edit Task' : 'Add Task')

@section('styles')
<style>
  .error-msg { color: red; font-size: 0.8rem; }
</style>
@endsection

@section('content')
@if (session('success'))
  <div class="success-msg">{{ session('success') }}</div>
@endif
````

---

## 表單開頭 + 欄位 1：Title

```blade
<form method="POST" action="{{ isset($task) ? route('tasks.update', $task->id) : route('tasks.store') }}">
  @csrf
  @isset($task) @method('PUT') @endisset

  <div>
    <label for="title">Title</label>
    <input type="text" name="title" value="{{ old('title', $task->title ?? '') }}">
    @error('title') <p class="error-msg">{{ $message }}</p> @enderror
  </div>
```

---

## 欄位 2 + 3：描述 / 長描述

```blade
  <div>
    <label for="description">Description</label>
    <textarea name="description" rows="5">{{ old('description', $task->description ?? '') }}</textarea>
    @error('description') <p class="error-msg">{{ $message }}</p> @enderror
  </div>

  <div>
    <label for="long_description">Long Description</label>
    <textarea name="long_description" rows="10">{{ old('long_description', $task->long_description ?? '') }}</textarea>
    @error('long_description') <p class="error-msg">{{ $message }}</p> @enderror
  </div>
```

---

## 提交按鈕 + 表單結尾

```blade
  <div>
    <button type="submit">
      {{ isset($task) ? 'Update Task' : 'Add Task' }}
    </button>
  </div>
</form>
```

---

## 刪除按鈕（只在編輯時顯示）

```blade
@isset($task)
<form method="POST" action="{{ route('tasks.destroy', $task->id) }}" style="margin-top: 1rem;">
  @csrf
  @method('DELETE')
  <button type="submit" onclick="return confirm('確定要刪除嗎？')">
    Delete Task
  </button>
</form>
@endisset

@endsection
```

---

## 建立任務頁：`tasks/create.blade.php`

```blade
@extends('layouts.app')

@section('content')
  @include('form')
@endsection
```

---

## 編輯任務頁：`tasks/edit.blade.php`

```blade
@extends('layouts.app')

@section('content')
  @include('form', ['task' => $task])
@endsection
```

---

## 路由設定：`routes/web.php`

```php
Route::resource('tasks', TaskController::class);
```

* 自動產生 `GET / POST / PUT / DELETE` 的 CRUD 路由
* 刪除功能對應 `DELETE /tasks/{id}`

---

## 刪除邏輯（TaskController）

```php
public function destroy(Task $task)
{
    $task->delete();

    return redirect()->route('tasks.index')
        ->with('success', '任務已成功刪除');
}
```

---

## 📌 小結

| 功能        | 寫法 / 概念                         |
| --------- | ------------------------------- |
| 表單共用      | `@include('form')` + `$task` 判斷 |
| 錯誤處理      | `@error()` + `old()`            |
| Method 切換 | `@method('PUT')` / `DELETE`     |
| 路由設計      | `Route::resource('tasks', ...)` |
| 資安保護      | `@csrf` + Laravel 表單驗證機制        |

---

## ✅ 重點回顧

* ✅ 建立一份「共用」任務表單模板（Create/Edit 都能用）
* ✅ 根據 `$task` 判斷是新增還是編輯（POST/PUT）
* ✅ 透過 `Route::resource()` 簡化 CRUD 路由
* ✅ 加入 `DELETE` 表單，實作刪除任務功能




