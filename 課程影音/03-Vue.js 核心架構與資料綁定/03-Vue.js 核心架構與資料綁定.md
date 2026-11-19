[🏠 回到首頁](../../README.md)

# 03-Vue.js 核心架構與資料綁定

## 3.1 MVVM 架構

### 1. MVVM 是什麼的縮寫？

| 縮寫 | 英文全名 | 中文 | 負責什麼 |
| --- | --- | --- | --- |
| **M** | Model | 資料 | 儲存應用程式的資料（例如 `name = "Anna"`） |
| **V** | View | 畫面 | 使用者看到、操作的畫面（HTML 模板） |
| **VM** | ViewModel | 橋樑 | 連接 View 與 Model，讓兩邊保持同步 |

**MVVM 的意義：** 把「資料」和「畫面」分開，中間由 ViewModel 當橋樑自動同步。

> 💡 比喻：餐廳裡，**View** 是客人看到的菜單與餐桌，**Model** 是廚房裡的食材與庫存，**ViewModel** 是服務生：客人點餐（操作畫面）→ 服務生通知廚房（更新資料）；廚房備好餐（資料變化）→ 服務生送上桌（更新畫面）。

![示意圖](MVVM.PNG)

### 2. 資料如何流動？（以輸入姓名為例）

```
使用者在輸入框打字
   │  ① 輸入 / 操作
   ▼
 View（畫面）──▶ ViewModel（橋樑，v-model）──② 更新資料──▶ Model（name）
   ▲                                                         │
   └──────────── ④ 更新畫面 ◀── ViewModel ◀── ③ 資料變化 ────┘
```

1. 使用者在輸入框輸入姓名（View）。
2. `v-model` 接收輸入，更新 `name` 這筆資料（Model）。
3. `name` 改變了，Vue 偵測到資料變化。
4. Vue 自動更新畫面，「你好，Anna！」跟著變（View）。

重點：**我們不需要自己寫「找到元素、改 innerText」的程式**，只要改資料，畫面就會自動更新。這就是**資料驅動畫面**。

### 3. 實作：MVVM.html

使用 Composition API，輸入姓名後，畫面即時顯示問候語。

```html
<div id="app">
  <!-- View：畫面 -->
  <label>姓名：<input type="text" v-model="name"></label>
  <p>你好，{{ name }}！</p>
</div>

<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  const { createApp, ref } = Vue;

  createApp({
    setup() {
      // Model：資料。ref() 會把它變成「響應式資料」
      const name = ref('Anna');

      // ViewModel：把資料交給模板使用
      return { name };
    }
  }).mount('#app');
</script>
```

程式碼說明：

| 程式碼 | 說明 |
| --- | --- |
| `ref('Anna')` | 建立響應式資料，初始值為 `'Anna'`。資料一變，使用它的畫面就會自動更新 |
| `setup()` | Composition API 的入口，在這裡宣告資料與函式 |
| `return { name }` | 把資料「交出去」，模板才能使用 `name` |
| `v-model="name"` | **雙向綁定**：輸入框改變 → `name` 改變；`name` 改變 → 輸入框也改變 |
| `{{ name }}` | 插值語法，把 `name` 的內容顯示在畫面上 |
| `.mount('#app')` | 告訴 Vue 要管理哪一塊 HTML |

> ⚠️ 執行方式：請在 VS Code 對 `MVVM.html` 按右鍵 → **Open with Live Server**。

> 💡 小提醒：在 `<script>` 中讀寫 `ref` 的值要用 `name.value`；在模板（HTML）中則直接寫 `name`，Vue 會自動處理。

## 3.2 Vue起手式

### 1. 何謂 Vue 元件？

**元件（Component）** 是「畫面的一小塊 + 它需要的資料與功能」包成的一個獨立單位。

> 💡 比喻：元件就像樂高積木。一個網頁由很多積木（按鈕、清單、導覽列…）組成，每塊積木各自負責自己的外觀與行為，可以重複使用。

本章的 `createApp({ ... })` 就是在建立最基本的元件（根元件）。

### 2. Vue 元件可拆成三個部分

| 部分 | 要回答的問題 | 做法 | 計數器範例 |
| --- | --- | --- | --- |
| 1. 資料 | 畫面需要使用哪些資料？ | 用 `ref()`、`reactive()` 建立響應式資料 | `count`（目前數字） |
| 2. 功能 | 使用者操作時需要做什麼？ | 建立 JavaScript 函式 | `add()`：將數字加 1 |
| 3. 生命週期 | 元件在不同階段要做什麼？ | 使用生命週期函式 | `onMounted()`：顯示到畫面後執行 |

**1. 資料**
- `ref()`：適合單一值（數字、字串、布林），讀寫要用 `.value`。
- `reactive()`：適合物件，直接用屬性存取，不需要 `.value`。
- 兩者建立的資料都是「響應式」：資料改變，畫面自動更新。

**2. 功能**
- 就是一般的 JavaScript 函式，再用 `@click="add"` 綁定到按鈕。
- `@click` 是 `v-on:click` 的縮寫，意思是「點擊時執行」。

**3. 生命週期**

生命週期是指 Vue 元件從**建立、顯示、更新到最後被移除**所經歷的整個流程。
Vue 提供對應的生命週期函式，讓我們在元件進入特定階段時，執行需要的程式。

```
建立 ──▶ 顯示到畫面（onMounted）──▶ 資料改變、更新畫面（onUpdated）──▶ 移除（onUnmounted）
```

| 生命週期函式 | 執行時機 | 常見用途 |
| --- | --- | --- |
| `onMounted()` | 元件顯示到畫面**之後** | 呼叫 API 取得資料、操作 DOM |
| `onUpdated()` | 畫面因資料改變而更新之後 | 少用，初學先認識即可 |
| `onUnmounted()` | 元件被移除之後 | 清除計時器、取消事件監聽 |

本節先會用 `onMounted()` 即可。

### 3. 實作：counter-vue.html（計數器）

```html
<div id="app">
  <h1>計數器</h1>
  <p>目前數字：{{ count }}</p>
  <button type="button" @click="add">+1</button>
</div>

<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  const { createApp, ref, onMounted } = Vue;

  createApp({
    setup() {
      // 1. 資料
      const count = ref(0);

      // 2. 功能：數字加 1
      const add = () => {
        count.value++;
      };

      // 3. 生命週期：元件顯示到畫面後執行
      onMounted(() => {
        console.log('元件已顯示到畫面上');
      });

      return { count, add };
    }
  }).mount('#app');
</script>
```

執行流程：

1. 頁面載入，Vue 建立元件，`count` 為 `0`，畫面顯示「目前數字：0」。
2. 元件顯示到畫面後，執行 `onMounted()`，主控台印出訊息（按 F12 → Console 可看到）。
3. 使用者按下按鈕 → 執行 `add()` → `count.value` 變成 `1`。
4. 因為 `count` 是響應式資料，畫面自動更新為「目前數字：1」。

> ⚠️ 執行方式：請在 VS Code 對 `counter-vue.html` 按右鍵 → **Open with Live Server**。

### 4. 常見錯誤

| 錯誤 | 原因 | 解法 |
| --- | --- | --- |
| 畫面顯示 `{{ count }}` 原文 | Vue 沒載入成功，或 `mount('#app')` 找不到元素 | 檢查 CDN 網址與 `id="app"` |
| 點按鈕數字不變 | 在 `<script>` 裡寫成 `count++`，忘了 `.value` | 改成 `count.value++` |
| 模板出現 `count is not defined` | 忘了在 `setup()` 中 `return` | 加上 `return { count, add }` |

## 3.3 單筆資料呈現與雙向資料綁定
