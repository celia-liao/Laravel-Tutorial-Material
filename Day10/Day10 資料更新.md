---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
marp: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')

---

# Laravel 表單更新功能

### 資料更新（Edit / Update）

---

## 🎯 今天要完成什麼？

- 建立任務編輯畫面 edit.blade.php
- 表單自動保留舊值（`old()`）
- 顯示欄位錯誤訊息（`@error`）
- 設定 update 路由與資料驗證
- 使用 Flash Message 呈現更新成功訊息

---

## 1️⃣ 建立任務編輯頁 edit.blade.php

```blade
<form method="POST" action="{{ route('tasks.update', ['id' => $task->id]) }}">
  @csrf
  @method('PUT')

  <input type="text" name="title" value="{{ old('title', $task->title) }}" />
  @error('title') <p class="error-msg">{{ $message }}</p> @enderror

  <textarea name="description">{{ old('description', $task->description) }}</textarea>
  @error('description') <p class="error-msg">{{ $message }}</p> @enderror

  <textarea name="long_description">{{ old('long_description', $task->long_description) }}</textarea>
  @error('long_description') <p class="error-msg">{{ $message }}</p> @enderror

  <button type="submit">Edit Task</button>
</form>
````

---

## 2️⃣ `old()` 是什麼？

```blade
value="{{ old('title', $task->title) }}"
```

* 驗證失敗：顯示剛剛輸入的內容
* 初次進入：顯示任務原本資料

適用於 `<input>`、`<textarea>` 欄位

---

## 3️⃣ 路由設定（web.php）

```php
// 顯示編輯頁面
Route::get('/tasks/{id}/edit', function ($id) {
    return view('edit', ['task' => Task::findOrFail($id)]);
})->name('tasks.edit');

// 接收更新資料
Route::put('/tasks/{id}', function ($id, Request $request) {
    $data = $request->validate([
        'title' => 'required|max:255',
        'description' => 'required',
        'long_description' => 'required',
    ]);

    $task = Task::findOrFail($id);
    $task->update($data);

    return redirect()->route('tasks.show', ['id' => $task->id])
        ->with('success', 'Task updated successfully');
})->name('tasks.update');
```

---

## 4️⃣ Flash Message 是什麼？

**Flash Message（閃現訊息）**：

* 只在「下次請求」中出現一次的通知訊息
* 通常用來顯示：建立成功、刪除成功、更新成功等
* Laravel 使用 session 傳遞這類訊息

---

## 實作：顯示成功訊息

```php
// Controller 中儲存訊息
return redirect()->route('tasks.show', ['id' => $task->id])
    ->with('success', '任務更新成功！');
```

```blade
{{-- Blade 中顯示訊息 --}}
@if (session()->has('success'))
  <div class="success-msg">{{ session('success') }}</div>
@endif
```

---

## 5️⃣ 總整理：Laravel 更新任務流程

| 步驟                          | 說明             |
| --------------------------- | -------------- |
| GET /tasks/{id}/edit        | 顯示任務編輯畫面，填入舊資料 |
| PUT /tasks/{id}             | 接收表單，進行資料驗證與更新 |
| 表單使用 @csrf + @method('PUT') | 保證安全與正確路由      |
| 使用 old(), @error            | 顯示舊資料與錯誤訊息     |
| 使用 ->with() + session()     | 顯示成功通知（Flash）  |

---

## ✅ 小結

* 使用 `old()` 與 `@error` 提升 UX，不用重填表單
* Flash Message 呈現操作結果，讓使用者清楚知道狀態
* 表單驗證、樣式管理與資料更新都一併學會 

