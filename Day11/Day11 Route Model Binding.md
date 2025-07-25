---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
marp: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')

---

# Laravel Route Model Binding  

---

## 🚨 問題背景

### 傳統做法遇到的問題

在 Laravel 中，我們經常需要：

```php
// ❌ 問題 1：每次都要手動查詢資料
Route::get('/tasks/{id}/edit', function ($id) {
    $task = Task::findOrFail($id);  // 重複的查詢邏輯
    return view('edit', ['task' => $task]);
});

// ❌ 問題 2：驗證邏輯散落在各處
Route::post('/tasks', function (Request $request) {
    $data = $request->validate([     // 驗證邏輯重複
        'title' => 'required|max:255',
        'description' => 'required',
        'long_description' => 'required',
    ]);
    Task::create($data);
});

// ❌ 問題 3：安全性問題
// 沒有保護機制，可能被惡意資料攻擊
```

### 解決方案

使用 Laravel 的兩個功能：
- **Route Model Binding**：自動查詢資料
- **Form Request**：集中驗證邏輯

---

## Route Model Binding

### 什麼是 Route Model Binding？

Laravel 會根據 URL 參數自動查詢對應的資料庫記錄。

### 基本用法

```php
// ✅ 使用 Route Model Binding
Route::get('/tasks/{task}', function (Task $task) {
    return view('show', compact('task'));
});
```

### 工作原理

當訪問 `/tasks/5` 時：
1. Laravel 看到 `{task}` 參數
2. 自動執行 `Task::findOrFail(5)`
3. 如果找不到資料，自動返回 404 錯誤

### 好處

- ✅ **簡潔**：不用手動寫查詢邏輯
- ✅ **安全**：自動處理找不到資料的情況
- ✅ **一致**：所有路由都使用相同模式

---

## Form Request

### 什麼是 Form Request？

將驗證邏輯封裝在專門的類別中，讓控制器更簡潔。

### 建立 Form Request

```bash
php artisan make:request TaskRequest
```

### 定義驗證規則

```php
// app/Http/Requests/TaskRequest.php
class TaskRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;  // 允許所有請求
    }

    public function rules(): array
    {
        return [
            'title' => 'required|max:255',
            'description' => 'required',
            'long_description' => 'required',
        ];
    }
}
```

### 在路由中使用

```php
Route::post('/tasks', function (TaskRequest $request) {
    // 自動驗證，失敗會自動返回錯誤頁面
    Task::create($request->validated());
});
```

---

## 🎯 `validated()` 方法

### 什麼是 `validated()`？

只返回通過驗證的資料，過濾掉未驗證的欄位。

### 範例

```php
// 假設表單提交了這些資料：
[
    'title' => '數學作業',
    'description' => '完成第三章習題',
    'long_description' => '詳細說明...',
    'hack_attempt' => '惡意資料'  // 未在驗證規則中定義
]

// $request->validated() 只會返回：
[
    'title' => '數學作業',
    'description' => '完成第三章習題', 
    'long_description' => '詳細說明...'
]
// 'hack_attempt' 被過濾掉了！
```

---

## Mass Assignment Protection

### 什麼是 Mass Assignment？

Laravel 為了安全，預設不允許批量寫入資料庫。

### 錯誤範例

```php
// 這樣會出錯：
Task::create([
    'title' => '作業',
    'description' => '說明'
]);
// 錯誤：MassAssignmentException
```

### 解決方法

在 Model 中設定 `$fillable`：

```php
// app/Models/Task.php
class Task extends Model
{
    protected $fillable = [
        'title',
        'description', 
        'long_description'
    ];
}
```

---

## 實際改造過程

### 步驟 1：建立 Form Request

```bash
php artisan make:request TaskRequest
```

### 步驟 2：設定驗證規則

```php
// app/Http/Requests/TaskRequest.php
public function rules(): array
{
    return [
        'title' => 'required|max:255',
        'description' => 'required',
        'long_description' => 'required',
    ];
}
```

### 步驟 3：更新路由

```php
// routes/web.php
use App\Http\Requests\TaskRequest;

// 顯示任務詳情
Route::get('/tasks/{task}', function (Task $task) {
    return view('show', compact('task'));
})->name('tasks.show');

// 顯示編輯表單
Route::get('/tasks/{task}/edit', function (Task $task) {
    return view('edit', compact('task'));
})->name('tasks.edit');

// 處理新增表單
Route::post('/tasks', function (TaskRequest $request) {
    Task::create($request->validated());
    return redirect()->route('tasks.index')
        ->with('success', '任務新增成功！');
})->name('tasks.store');

// 處理更新表單
Route::put('/tasks/{task}', function (Task $task, TaskRequest $request) {
    $task->update($request->validated());
    return redirect()->route('tasks.show', ['task' => $task])
        ->with('success', '任務更新成功！');
})->name('tasks.update');
```

### 步驟 4：更新視圖

```php
// resources/views/index.blade.php
<a href="{{ route('tasks.show', ['task' => $task]) }}">
    {{ $task->title }}
</a>

// resources/views/edit.blade.php
<form method="POST" action="{{ route('tasks.update', ['task' => $task]) }}">
    @csrf
    @method('PUT')
    <!-- 表單內容 -->
</form>
```

---

## 📊 完整代碼對比

### ❌ 改造前

```php
// 路由：需要手動查詢
Route::get('/tasks/{id}/edit', function ($id) {
    $task = Task::findOrFail($id);
    return view('edit', ['task' => $task]);
});

// 處理表單：需要手動驗證
Route::put('/tasks/{id}', function ($id, Request $request) {
    $data = $request->validate([
        'title' => 'required|max:255',
        'description' => 'required',
        'long_description' => 'required',
    ]);
    
    $task = Task::findOrFail($id);
    $task->update($data);
    
    return redirect()->route('tasks.show', ['id' => $task->id]);
});
```

### ✅ 改造後

```php
// 路由：自動查詢
Route::get('/tasks/{task}/edit', function (Task $task) {
    return view('edit', compact('task'));
});

// 處理表單：自動驗證
Route::put('/tasks/{task}', function (Task $task, TaskRequest $request) {
    $task->update($request->validated());
    
    return redirect()->route('tasks.show', ['task' => $task]);
});
```

---

## ⚠️ 常見錯誤與解決

### 錯誤 1：Missing required parameter

**錯誤訊息：**
```
Missing required parameter for [Route: tasks.show] [URI: tasks/{task}] [Missing parameter: task].
```

**原因：** 重定向時沒有正確傳遞參數

**解決方法：**
```php
// ❌ 錯誤
return redirect()->route('tasks.show', $task);

// ✅ 正確
return redirect()->route('tasks.show', ['task' => $task]);
```

### 錯誤 2：MassAssignmentException

**錯誤訊息：**
```
MassAssignmentException
```

**原因：** Model 沒有設定 `$fillable`

**解決方法：**
```php
// app/Models/Task.php
class Task extends Model
{
    protected $fillable = [
        'title',
        'description',
        'long_description'
    ];
}
```

### 錯誤 3：驗證失敗時沒有錯誤訊息

**原因：** Form Request 的 `authorize()` 方法返回 `false`

**解決方法：**
```php
public function authorize(): bool
{
    return true;  // 改為 true
}
```

---

## 🎯 小結

### 學到的技能

| 功能 | 改造前 | 改造後 | 好處 |
|------|--------|--------|------|
| **查詢資料** | `Task::findOrFail($id)` | `Task $task` | 更簡潔、自動處理錯誤 |
| **驗證資料** | 在路由中手動驗證 | `TaskRequest` | 邏輯集中、可重用 |
| **儲存資料** | `$task->update($data)` | `$task->update($request->validated())` | 更安全、只儲存驗證過的資料 |
| **安全性** | 手動設定 `$fillable` | 自動過濾未授權欄位 | 防止惡意資料寫入 |

