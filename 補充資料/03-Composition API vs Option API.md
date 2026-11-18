[🏠 回到首頁](../README.md)

# 03-Composition API vs Option API

## 一、Option API 與 Composition API 的比較

### 1-1 意義

Vue 3 提供兩種撰寫元件邏輯的風格，兩者**可以達成相同的功能**，差別在於「程式碼如何組織」。

| 項目     | Option API（選項式 API）                                          | Composition API（組合式 API）                    |
| -------- | ----------------------------------------------------------------- | ------------------------------------------------ |
| 出現時間 | Vue 2 起就是標準寫法，Vue 3 繼續支援                              | **Vue 3（2020）** 新增，受 React Hooks 啟發      |
| 組織方式 | 依「**選項類型**」分類：`data`、`methods`、`computed`、`mounted`… | 依「**功能邏輯**」組合：相關的狀態與函式寫在一起 |
| 進入點   | 物件選項（`data()`、`methods`…）                                  | `setup()` 或 `<script setup>`                    |
| 響應式   | 回傳的 `data()` 自動變成響應式                                    | 需明確宣告：`ref()`、`reactive()`                |
| 取得資料 | 透過 `this`                                                       | 直接使用變數（沒有 `this`）                      |
| 生命週期 | `mounted()`、`created()`…（選項）                                 | `onMounted()`、`onUnmounted()`…（函式）          |
| 官方定位 | 適合入門與簡單元件                                                | 官方**推薦**，特別是搭配 TypeScript 與大型專案   |

> 一句話：**Option API 是「按類別分抽屜」；Composition API 是「按功能整理」。**

### 1-2 寫法對照

同一個計數器，兩種寫法：

```html
<div id="app">
  <button @click="add">{{ count }}</button>
</div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  // ===== Option API =====
  Vue.createApp({
    data() {
      return { count: 0 }; // 狀態放在 data
    },
    methods: {
      add() {
        this.count++; // 用 this 存取 data
      },
    },
  }).mount("#app");

  // ===== Composition API =====
  const { ref } = Vue;
  Vue.createApp({
    setup() {
      const count = ref(0); // 狀態：用 ref 宣告為響應式
      const add = () => {
        count.value++; // JS 中要用 .value 存取
      };
      return { count, add }; // 要回傳，模板才能使用
    },
  }).mount("#app");
</script>
```

在單檔元件（SFC，`.vue` 檔，搭配 Vite）中，Composition API 通常使用更簡潔的 `<script setup>`：

```vue
<script setup>
import { ref } from "vue";
const count = ref(0); // 不用 return，頂層宣告的變數模板都能直接用
const add = () => count.value++;
</script>

<template>
  <button @click="add">{{ count }}</button>
</template>
```

### 1-3 特色比較

| 比較面向     | Option API                                         | Composition API                                                   |
| ------------ | -------------------------------------------------- | ----------------------------------------------------------------- |
| 學習曲線     | **平緩**，結構固定、容易知道東西放哪               | 稍陡，需理解 `ref` / `reactive`、`.value`、閉包                   |
| 程式碼組織   | 同一功能被拆散在 `data`、`methods`、`mounted` 之間 | **同一功能集中**，閱讀時不必上下跳                                |
| 邏輯重用     | `mixins`（命名衝突、來源不明）                     | **composables**（自訂函式如 `useFetch()`），清楚且可組合          |
| TypeScript   | 型別推導較弱，`this` 型別複雜                      | **型別推導良好**，是普通變數與函式                                |
| `this`       | 必須使用，箭頭函式中易出錯                         | 沒有 `this`，沒有指向問題                                         |
| 樣板碼       | 少，結構固定                                       | `setup()` 要 `return`；`<script setup>` 則更簡潔                  |
| CDN 直接使用 | 非常適合                                           | 可以，但 `setup()` 要 return，較冗長；`<script setup>` 需建置工具 |
| 打包體積     | 較大（整個選項系統）                               | 較易 tree-shaking，體積較小                                       |

#### 邏輯組織的差異

```
Option API：功能 A、B 的程式碼被切散           Composition API：功能 A、B 各自集中
┌────────────────────┐                        ┌────────────────────┐
│ data    → A、B     │                        │ // 功能 A           │
│ methods → A、B     │                        │ 狀態 + 函式 + 生命週期│
│ mounted → A、B     │                        │ // 功能 B           │
└────────────────────┘                        │ 狀態 + 函式 + 生命週期│
                                              └────────────────────┘
```

### 1-4 優缺點與應用

**Option API**

- 優：結構固定、新手容易上手；適合 CDN 與漸進式導入；舊專案（Vue 2）相容。
- 缺：元件變大後，同一功能的程式碼散落各處；邏輯重用困難（mixins）；TypeScript 支援較弱。

**Composition API**

- 優：邏輯集中、可抽成 composables 重用；TypeScript 友善；沒有 `this`；與 Pinia、VueUse 等生態契合。
- 缺：初學較抽象；需要分清 `ref`（`.value`）與 `reactive`；寫法自由，團隊容易風格不一，需要規範。

**應用建議**

| 情境                                         | 建議                                    |
| -------------------------------------------- | --------------------------------------- |
| 教學、CDN 小頁面、舊專案維護                 | Option API                              |
| 新專案、Vite + SFC、TypeScript               | **Composition API（`<script setup>`）** |
| 大型元件、多處共用的邏輯（如請求、表單驗證） | Composition API + composables           |
| 團隊成員剛接觸 Vue                           | 先 Option API，再逐步轉 Composition API |

> 兩者可以**同一專案並存**，甚至同一元件混用（不建議）。官方文件目前以 Composition API 為主要推薦。

---

## 二、從原生 JS 逐步轉成 Option API 與 Composition API

以兩個範例說明，並與 [02-三大框架比較.md](02-三大框架比較.md) 的 Vue 寫法呼應：

1. `getProducts.html`：呼叫 API 並渲染清單
2. `todolist.html`：新增 / 刪除待辦

### 2-1 範例一：getProducts（串接 API 取得產品資料）

#### ① 原生 JS（原始碼）

```html
<ul id="list"></ul>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>
  const api_path = "kentlee406";
  axios
    .get(
      `https://livejs-api.hexschool.io/api/livejs/v1/customer/${api_path}/products`,
    )
    .then((response) => {
      const list = document.getElementById("list");
      response.data.products.forEach((product) => {
        const li = document.createElement("li");
        li.textContent = product.title;
        list.appendChild(li);
      });
    })
    .catch((error) => console.error(error));
</script>
```

#### ② 轉成 Option API（逐步）

**步驟一：建立環境**

```html
<div id="app"></div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>
  // 建立 Vue 應用並掛載到 #app
  Vue.createApp({}).mount("#app");
</script>
```

**步驟二：在 `data` 宣告狀態**

```html
<script>
  Vue.createApp({
    // data：存放狀態，Vue 會自動把它變成響應式
    data() {
      return {
        apiPath: "kentlee406", // 對應原本的 api_path
        products: [], // 產品清單，先放空陣列
      };
    },
  }).mount("#app");
</script>
```

**步驟三：撰寫模板**

```html
<div id="app">
  <ul>
    <!-- v-for 取代 createElement + appendChild；:key 供 Vue 識別項目 -->
    <li v-for="product in products" :key="product.id">{{ product.title }}</li>
  </ul>
</div>
```

**步驟四：把請求放進 `methods`，並在 `mounted()` 呼叫**

```html
<div id="app">
  <ul>
    <li v-for="product in products" :key="product.id">{{ product.title }}</li>
  </ul>
</div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>
  Vue.createApp({
    data() {
      return {
        apiPath: "kentlee406",
        products: [],
      };
    },
    // methods：放函式。用 this 存取 data
    methods: {
      getProducts() {
        axios
          .get(
            `https://livejs-api.hexschool.io/api/livejs/v1/customer/${this.apiPath}/products`,
          )
          .then((res) => {
            // 改資料即可，畫面自動更新（用箭頭函式，this 才會指向元件）
            this.products = res.data.products;
          })
          .catch((error) => console.error(error));
      },
    },
    // mounted：生命週期選項，畫面掛載後自動執行
    mounted() {
      this.getProducts();
    },
  }).mount("#app");
</script>
```

#### ③ 轉成 Composition API（逐步）

**步驟一：建立環境與 `setup()`**

```html
<div id="app"></div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>
  // 從全域 Vue 取出需要的函式（有建置工具時則 import { ref, onMounted } from "vue"）
  const { createApp, ref, onMounted } = Vue;

  createApp({
    // setup：Composition API 的進入點，元件建立時執行一次
    setup() {
      return {};
    },
  }).mount("#app");
</script>
```

**步驟二：用 `ref` 宣告狀態**

```html
<script>
  const { createApp, ref, onMounted } = Vue;

  createApp({
    setup() {
      const apiPath = "kentlee406"; // 不會改變的值，一般常數即可
      // ref：建立響應式資料。在 JS 中用 .value 存取，模板中會自動解開
      const products = ref([]);

      // 回傳給模板使用的東西
      return { products };
    },
  }).mount("#app");
</script>
```

**步驟三：撰寫模板**（與 Option API 完全相同）

```html
<div id="app">
  <ul>
    <li v-for="product in products" :key="product.id">{{ product.title }}</li>
  </ul>
</div>
```

**步驟四：定義函式並在 `onMounted()` 呼叫**

```html
<div id="app">
  <ul>
    <li v-for="product in products" :key="product.id">{{ product.title }}</li>
  </ul>
</div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>
  const { createApp, ref, onMounted } = Vue;

  createApp({
    setup() {
      const apiPath = "kentlee406";
      const products = ref([]); // 狀態

      // 函式就是普通函式，不需要 this
      const getProducts = () => {
        axios
          .get(
            `https://livejs-api.hexschool.io/api/livejs/v1/customer/${apiPath}/products`,
          )
          .then((res) => {
            // ref 在 JS 中要寫 .value
            products.value = res.data.products;
          })
          .catch((error) => console.error(error));
      };

      // onMounted：生命週期「函式」，掛載後執行（取代 mounted 選項）
      onMounted(() => {
        getProducts();
      });

      // 要提供給模板的狀態與函式都需 return
      return { products };
    },
  }).mount("#app");
</script>
```

> 使用 Vite + SFC 時可改寫成 `<script setup>`：
>
> ```vue
> <script setup>
> import { ref, onMounted } from "vue";
> import axios from "axios";
>
> const apiPath = "kentlee406";
> const products = ref([]);
>
> const getProducts = async () => {
>   const res = await axios.get(
>     `https://livejs-api.hexschool.io/api/livejs/v1/customer/${apiPath}/products`,
>   );
>   products.value = res.data.products;
> };
>
> onMounted(getProducts); // 不用 return
> </script>
> ```

#### ④ 範例一對照

| 步驟     | 原生 JS                         | Option API                | Composition API            |
| -------- | ------------------------------- | ------------------------- | -------------------------- |
| 存資料   | 無狀態                          | `data() { products: [] }` | `const products = ref([])` |
| 函式位置 | 直接寫在 script                 | `methods`                 | `setup()` 內的普通函式     |
| 存取資料 | —                               | `this.products`           | `products.value`           |
| 請求時機 | 直接執行                        | `mounted()`               | `onMounted()`              |
| 更新畫面 | `createElement` + `appendChild` | `this.products = ...`     | `products.value = ...`     |
| 渲染清單 | 手動建立 DOM                    | `v-for`（相同）           | `v-for`（相同）            |

---

### 2-2 範例二：todolist（新增 / 刪除待辦）

#### ① 原生 JS（原始碼）

```html
<input type="text" placeholder="請輸入代辦事項" id="txt" />
<button id="add">新增</button>
<ul id="list"></ul>
<script>
  const data = [];
  let nextId = 1;
  const txt = document.getElementById("txt");
  const add = document.getElementById("add");
  const list = document.getElementById("list");
  let listHTML = "";

  function renderList() {
    listHTML = "";
    data.forEach((item) => {
      listHTML += `<li>${item.message} <button class="delete" data-id="${item.id}">刪除</button></li>`;
    });
    list.innerHTML = listHTML;
  }

  // 新增代辦
  add.addEventListener("click", () => {
    const id = nextId++;
    const message = txt.value.trim();
    if (!message) return;
    data.push({ id, message });
    txt.value = "";
    renderList();
  });

  // 刪除代辦（事件委派）
  list.addEventListener("click", (e) => {
    if (e.target.classList.contains("delete")) {
      const id = Number(e.target.dataset.id);
      const index = data.findIndex((item) => item.id === id);
      if (index !== -1) {
        data.splice(index, 1);
        renderList();
      }
    }
  });
</script>
```

#### ② 轉成 Option API（逐步）

**步驟一：建立環境**

```html
<div id="app"></div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  Vue.createApp({}).mount("#app");
</script>
```

**步驟二：宣告 `data` 與 `methods`**

```html
<script>
  Vue.createApp({
    // data：取代全域變數
    data() {
      return {
        data: [], // 待辦清單，格式 { id, message }
        text: "", // 輸入框內容（取代 txt.value）
        nextId: 1, // 下一個 id
      };
    },
    // methods：取代事件處理函式，先建立空殼
    methods: {
      addTodo() {},
      deleteTodo(id) {},
    },
  }).mount("#app");
</script>
```

**步驟三：撰寫模板**

```html
<div id="app">
  <!-- v-model：雙向綁定，取代 txt.value 與 getElementById -->
  <input v-model="text" placeholder="輸入代辦事項" />
  <!-- @click：取代 addEventListener -->
  <button @click="addTodo">新增</button>
  <ul>
    <!-- v-for：取代 renderList() 與 innerHTML -->
    <li v-for="item in data" :key="item.id">
      {{ item.message }}
      <!-- 直接傳 id，不需事件委派與 dataset -->
      <button @click="deleteTodo(item.id)">刪除</button>
    </li>
  </ul>
</div>
```

**步驟四：完成 `addTodo()` 與 `deleteTodo(id)`**

```html
<div id="app">
  <input v-model="text" placeholder="輸入代辦事項" @keyup.enter="addTodo" />
  <button @click="addTodo">新增</button>
  <ul>
    <li v-for="item in data" :key="item.id">
      {{ item.message }}
      <button @click="deleteTodo(item.id)">刪除</button>
    </li>
  </ul>
</div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  Vue.createApp({
    data() {
      return { data: [], text: "", nextId: 1 };
    },
    methods: {
      // 新增待辦
      addTodo() {
        const message = this.text.trim(); // 用 this 存取 data
        if (!message) return; // 空白不新增
        this.data.push({ id: this.nextId++, message });
        this.text = ""; // 清空輸入框
      },
      // 刪除待辦
      deleteTodo(id) {
        this.data = this.data.filter((item) => item.id !== id);
      },
    },
  }).mount("#app");
</script>
```

#### ③ 轉成 Composition API（逐步）

**步驟一：建立環境與 `setup()`**

```html
<div id="app"></div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  const { createApp, ref } = Vue;

  createApp({
    setup() {
      return {};
    },
  }).mount("#app");
</script>
```

**步驟二：用 `ref` 宣告狀態、建立函式空殼**

```html
<script>
  const { createApp, ref } = Vue;

  createApp({
    setup() {
      // 每個狀態各用一個 ref
      const todos = ref([]); // 待辦清單，格式 { id, message }
      const text = ref(""); // 輸入框內容
      let nextId = 1; // 不需要響應式的值，一般變數即可（不影響畫面）

      // 函式空殼
      const addTodo = () => {};
      const deleteTodo = (id) => {};

      // 模板會用到的都要回傳
      return { todos, text, addTodo, deleteTodo };
    },
  }).mount("#app");
</script>
```

**步驟三：撰寫模板**（與 Option API 相同，`ref` 在模板中會自動解開，不用寫 `.value`）

```html
<div id="app">
  <input v-model="text" placeholder="輸入代辦事項" />
  <button @click="addTodo">新增</button>
  <ul>
    <li v-for="item in todos" :key="item.id">
      {{ item.message }}
      <button @click="deleteTodo(item.id)">刪除</button>
    </li>
  </ul>
</div>
```

**步驟四：完成 `addTodo` 與 `deleteTodo`**

```html
<div id="app">
  <input v-model="text" placeholder="輸入代辦事項" @keyup.enter="addTodo" />
  <button @click="addTodo">新增</button>
  <ul>
    <li v-for="item in todos" :key="item.id">
      {{ item.message }}
      <button @click="deleteTodo(item.id)">刪除</button>
    </li>
  </ul>
</div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  const { createApp, ref } = Vue;

  createApp({
    setup() {
      const todos = ref([]);
      const text = ref("");
      let nextId = 1;

      // 新增待辦
      const addTodo = () => {
        const message = text.value.trim(); // 在 JS 中，ref 要加 .value
        if (!message) return;
        todos.value.push({ id: nextId++, message });
        text.value = ""; // 清空輸入框
      };

      // 刪除待辦
      const deleteTodo = (id) => {
        todos.value = todos.value.filter((item) => item.id !== id);
      };

      return { todos, text, addTodo, deleteTodo };
    },
  }).mount("#app");
</script>
```

> 使用 Vite + SFC 時可改寫成 `<script setup>`：
>
> ```vue
> <script setup>
> import { ref } from "vue";
>
> const todos = ref([]);
> const text = ref("");
> let nextId = 1;
>
> const addTodo = () => {
>   const message = text.value.trim();
>   if (!message) return;
>   todos.value.push({ id: nextId++, message });
>   text.value = "";
> };
>
> const deleteTodo = (id) => {
>   todos.value = todos.value.filter((item) => item.id !== id);
> };
> </script>
> ```

#### ④ 範例二對照

| 比較項目      | 原生 JS                       | Option API       | Composition API                         |
| ------------- | ----------------------------- | ---------------- | --------------------------------------- |
| 狀態宣告      | 全域變數                      | `data()`         | `ref()` / `reactive()`                  |
| 函式位置      | 全域函式 / 事件回呼           | `methods`        | `setup()` 內的普通函式                  |
| 存取狀態      | 直接用變數                    | `this.text`      | `text.value`（模板中免 `.value`）       |
| 輸入框        | 手動讀寫 `txt.value`          | `v-model="text"` | `v-model="text"`                        |
| 事件綁定      | `addEventListener` / 事件委派 | `@click`         | `@click`                                |
| 清單渲染      | `innerHTML` + `renderList()`  | `v-for`          | `v-for`                                 |
| 更新畫面      | **手動**                      | **自動**         | **自動**                                |
| 需要 `return` | —                             | 不需要           | `setup()` 需要；`<script setup>` 不需要 |

---

### 2-3 與原生 JS 的優缺點分析

#### 原生 JS

| 優點                                   | 缺點                                           |
| -------------------------------------- | ---------------------------------------------- |
| 零依賴、不需任何框架                   | 資料與畫面手動同步，漏掉 `renderList()` 就出錯 |
| 直接學到 DOM、事件、事件委派等底層原理 | `innerHTML` 拼接字串，效能差且有 XSS 風險      |
| 載入最快、小頁面最輕量                 | 功能一多，DOM 操作難以維護                     |

#### Option API

| 優點                                            | 缺點                                        |
| ----------------------------------------------- | ------------------------------------------- |
| 結構固定（`data` / `methods` / 生命週期），好學 | 必須處理 `this`（箭頭函式、回呼中容易指錯） |
| 與原生 JS 的思路接近，轉換容易                  | 同一功能散落在不同選項，元件大時難閱讀      |
| CDN 直接使用最簡單                              | 邏輯重用只能靠 mixins，易衝突、來源不明     |
|                                                 | TypeScript 支援較弱                         |

#### Composition API

| 優點                                                | 缺點                                      |
| --------------------------------------------------- | ----------------------------------------- |
| 同一功能的狀態、函式、生命週期寫在一起，可讀性高    | 需理解 `ref`、`.value`、`reactive` 的差別 |
| 沒有 `this`，函式就是普通函式                       | CDN 版需 `return`，較冗長                 |
| 可抽成 composables（如 `useTodos()`）重用邏輯       | 自由度高，團隊容易風格不一致              |
| TypeScript 型別推導良好；搭配 `<script setup>` 簡潔 | 初學者的抽象門檻較高                      |

#### 總結比較

| 考量面向         | 原生 JS        | Option API | Composition API                     |
| ---------------- | -------------- | ---------- | ----------------------------------- |
| 上手難度         | 低（但維護累） | ⭐ 低      | 中                                  |
| 程式碼量         | 多             | 少         | 少（`<script setup>` 最少）         |
| 維護性（大專案） | 差             | 中         | ⭐ 佳                               |
| 邏輯重用         | 差             | mixins     | ⭐ composables                      |
| TypeScript       | —              | 弱         | ⭐ 強                               |
| 是否需建置工具   | 否             | 否         | `setup()` 否；`<script setup>` 需要 |
| 官方推薦         | —              | 支援       | ⭐ 主要推薦                         |

#### 結論

1. 兩種 API 對原生 JS 的優勢相同：**資料驅動畫面，不用再手動管 DOM**（`v-model`、`v-for`、`@click`、響應式）。
2. Option API 與原生 JS 思路較近、結構固定，適合入門與 CDN 小頁面。
3. Composition API 讓邏輯依功能集中、容易重用，適合新專案、大型元件與 TypeScript。
4. 學習路徑建議：**原生 JS → Option API（理解響應式與指令）→ Composition API（學習 `ref`、composables、`<script setup>`）**。
5. 兩者模板語法完全相同，轉換時只需要改 `<script>` 的部分。
