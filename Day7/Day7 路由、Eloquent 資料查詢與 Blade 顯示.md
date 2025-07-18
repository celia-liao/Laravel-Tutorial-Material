---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
marp: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---
# Laravel Day 7

### 路由、Eloquent 資料查詢與 Blade 顯示

---

## 🎯 這次我們要完成什麼？

✅ 練習使用 Route 配合 Model 抓取資料 
✅ 讓 Blade 畫面自動讀取資料庫的內容
✅ 學會用 Eloquent 查詢語法進行資料查詢

---

## 什麼是 Eloquent？
`Eloquent` 是 Laravel 內建的 ORM（Object Relational Mapping，物件關聯對應）系統。

- 讓你「用物件」操作資料表
- 每個 Model 代表一張資料表
- 你可以用 Task::find(1) 或 Task::where('completed', true)->get()
這種「像在寫 PHP 物件」的語法來查詢、建立、更新、刪除資料！
#### 🎯重點：
- 不用手寫 SQL，大部分情境用 Eloquent 就能解決資料存取問題！

---


## 1. 任務列表頁：從資料庫抓出所有任務

<small>

```php
Route::get('/tasks', function () { 
    return view('index', [
        'tasks' => Task::latest()->get()
    ]);
})->name('tasks.index');
```


| 部分                      | 說明                                   |
| ----------------------- | ------------------------------------ |
| `Task::latest()->get()` | 使用 Eloquent 取出任務列表，依照時間由新到舊排序        |
| `'tasks' => ...`        | 傳資料到 Blade 模板中的 `$tasks` 變數          |
| `view('index', [...])`  | 載入 `resources/views/index.blade.php` |


</small>


---
##  2. 任務詳情頁：根據 id 抓出單筆資料

<small>

```php
Route::get('/tasks/{id}', function ($id) { 
    return view('show', ['task' => Task::findOrFail($id)]);
})->name('tasks.show');
```

| 部分                      | 說明                                  |
| ----------------------- | ----------------------------------- |
| `Task::findOrFail($id)` | 根據 ID 取得任務，找不到就會自動拋出 404            |
| `'task' => ...`         | 傳資料到 `show.blade.php` 中的 `$task` 變數 |

</small>


---

## 補充 | 常用查詢語法

| Eloquent 查詢                             | 說明                     |
| --------------------------------------- | ---------------------- |
| `Task::all()`                           | 取得所有任務                 |
| `Task::find($id)`                       | 找特定 ID 的任務（找不到會回 null） |
| `Task::findOrFail($id)`                 | 找不到就報錯（404）            |
| `Task::latest()->get()`                 | 依照 `created_at` 由新到舊排序 |
| `Task::where('completed', true)->get()` | 篩選完成的任務                |

---

##  🎯 小結

| 項目          | 功能                        |
| ----------- | ------------------------- |
| 路由          | 負責設定網址對應的邏輯               |
| Eloquent 模型 | 負責跟資料庫互動                  |
| Blade View  | 顯示畫面、接收從 controller 傳來的資料 |

---

## 📚 官方文件連結（推薦收藏）

🔗 Laravel 11.x 資料庫查詢文件（Eloquent + Query Builder）：
👉 [https://laravel.com/docs/11.x/queries](https://laravel.com/docs/11.x/queries)


