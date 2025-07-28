---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
marp: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')

---

# Laravel 專案中整合 Vite

---
### 📦 Vite 是什麼？

Vite 是一個現代前端建構工具，擁有以下優點：

* 開發時**極快的熱更新**（HMR）
* 使用原生 ES modules，不需繁重 bundle
* 支援 Vue、React 等框架
* Laravel 9 之後預設改用 Vite（取代 Mix）

---

### ✅ 為什麼要整合到 Laravel？

* Laravel 9 起預設使用 Vite
* 可以用 `@vite` 指令快速載入資源
* 若你要整合 Vue / React 或 Tailwind，也需要 Vite
* 管理 JS / CSS 更現代、更快、更簡單


---

### 🔧 步驟一：確認 Laravel 專案版本

```bash
php artisan --version
```

若版本為 Laravel 9.x 或以上，可使用 Vite。

---

### 🧰 步驟二：安裝 Vite 所需套件

```bash
npm install
```

如果你是從 Laravel 9+ 建立的專案，預設已經有 Vite config，檢查是否有這些檔案：

* `vite.config.js`
* `resources/js/app.js`
* `resources/css/app.css`

---

### 🔧 步驟三：設定 `vite.config.js`

確認內容如下（Laravel 預設）：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
  plugins: [
    laravel([
      'resources/css/app.css',
      'resources/js/app.js',
    ]),
  ],
});
```

---

### 🧱 步驟四：Blade 檔案中使用 Vite

在 `resources/views/layouts/app.blade.php` 中：

```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@yield('title')</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    @yield('content')
</body>
</html>
```

---

### 🧪 步驟五：啟動開發伺服器

```bash
npm run dev
```

這樣每次修改 JS / CSS，會自動熱更新（Hot Module Reloading）

---

### ✅ 補充：切換為正式環境（Production）

部署前輸出最終資源：

```bash
npm run build
```

生成的檔案會進入 `public/build`，供 Laravel 使用。

---

## 📌 小結：Laravel 整合 Vite 流程

| 步驟  | 說明                            |
| --- | ----------------------------- |
| 1️⃣ | 確認 Laravel 為 9.x 以上版本         |
| 2️⃣ | 使用 `npm install` 安裝 Vite 相關套件 |
| 3️⃣ | 確認 `vite.config.js` 有正確載入檔案   |
| 4️⃣ | Blade 中使用 `@vite()` 載入資源      |
| 5️⃣ | 使用 `npm run dev` 啟動前端開發       |
| 6️⃣ | 上線時使用 `npm run build` 進行打包    |

