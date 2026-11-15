# 將todolist從JS轉到Vue

只列body的部分

## JS程式碼

```
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
    const id = nextId++; // 遞增 id，刪除後也不會重複
    const message = txt.value.trim();
    if (!message) return;
    data.push({ id, message });
    txt.value = "";
    renderList();
  });

  // 刪除代辦
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

## 步驟一：建立 `todolist-vue.html` 的環境

載入CDN、新增#app、Vue.createApp({}).mount("#app");

```
<div id="app"></div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  Vue.createApp({}).mount("#app");
</script>
```

## 步驟二：建立data和methods

```
<div id="app"></div>
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  Vue.createApp({
    data() {
      return {
        data: [], // 代辦清單，格式 { id, message }
        text: "", // 輸入框內容
        nextId: 1, // 下一個 id
      };
    },
    methods: {
      addTodo() {},
      deleteTodo(id) {},
    },
  }).mount("#app");
</script>
```

## 步驟三：建立 `<div id="app">...</div>` 的內容

包括輸入框、新增按鈕和要渲染的資料

```
<div id="app">
  <input v-model="text" placeholder="輸入代辦事項" />
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
      return {
        data: [], // 代辦清單，格式 { id, message }
        text: "", // 輸入框內容
        nextId: 1, // 下一個 id
      };
    },
    methods: {
      addTodo() {},
      deleteTodo(id) {},
    },
  }).mount("#app");
</script>
```

## 步驟四：完成addTodo()和deleteTodo(id)

```
<div id="app">
<input v-model="text" placeholder="輸入代辦事項" />
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
    return {
      data: [], // 代辦清單，格式 { id, message }
      text: "", // 輸入框內容
      nextId: 1, // 下一個 id
    };
  },
  methods: {
    addTodo() {
      const message = this.text.trim();
      if (!message) return;
      this.data.push({ id: this.nextId++, message });
      this.text = "";
    },
    deleteTodo(id) {
      this.data = this.data.filter((item) => item.id !== id);
    },
  },
}).mount("#app");
</script>
```

## 步驟五：Open with Live Server

## 優缺點分析

### 一句話總結

原生 JS 是「**命令式**」：要自己告訴瀏覽器「怎麼做」（找元素 → 綁事件 → 組 HTML → 塞回畫面）。
Vue 是「**宣告式**」：只描述「資料長什麼樣子，畫面就長什麼樣子」，更新畫面的工作交給 Vue。

### 對照表

| 比較項目 | 轉換前（原生 JS） | 轉換後（Vue 3） |
| --- | --- | --- |
| 程式風格 | 命令式，手動操作 DOM | 宣告式，資料驅動畫面 |
| 取得元素 | 每個元素都要 `getElementById` | 不需要，用 `v-model`、`@click` 綁定 |
| 輸入框同步 | 手動讀 `txt.value`、清空 `txt.value = ""` | `v-model="text"` 雙向綁定，`this.text = ""` 即可 |
| 事件綁定 | `addEventListener`；刪除用事件委派（`e.target.classList`、`dataset.id`） | `@click="addTodo"`、`@click="deleteTodo(item.id)"` |
| 渲染清單 | 每次變動都要呼叫 `renderList()`，用字串拼接 `innerHTML` | `v-for` 自動渲染，改 `data` 畫面就更新 |
| 更新時機 | 必須記得手動呼叫 `renderList()`，忘了畫面就不會變 | 響應式系統自動偵測資料變化 |
| 程式碼量 | JS 約 30 行 | 邏輯約 10 行（`addTodo` + `deleteTodo`） |
| HTML 結構 | `<ul id="list">` 是空殼，內容藏在 JS 字串裡 | 模板直接寫在 HTML，結構一目了然 |
| 資料結構 | 物件 `{ id, message }`，`nextId` 遞增 | 物件 `{ id, message }`，`nextId` 遞增，`:key="item.id"` |
| 載入成本 | 零依賴 | 需載入 Vue（CDN 約數十 KB） |

### JS 版本

**優點**

- 零依賴，不需要網路或額外函式庫，打開瀏覽器就能跑。
- 能直接接觸 DOM API，有助於理解瀏覽器底層運作（事件、事件委派、`dataset`）。
- 沒有框架的載入與執行成本，小頁面最輕量。
- 資料用 `{ id, message }` 物件儲存，結構比較完整，適合日後擴充；`nextId` 計數器確保 id 不重複。

**缺點**

- 要手動維護「資料」與「畫面」同步，資料改了卻忘記呼叫 `renderList()` 就會不同步。
- 用 `innerHTML` 字串拼接：整個清單重繪，效能較差；若內容未處理，還有 **XSS 風險**（`${item.message}` 直接塞進 HTML）。
- 刪除按鈕若用重複的 `id="delete"` 違反 HTML 規範（`id` 必須唯一），因此改用 `class="delete"`。
- 刪除要用事件委派 + `dataset` + `Number()` 轉型 + `findIndex`，寫法繞，可讀性低。
- 原本的 `id = data.length + 1` 有 bug：刪除中間項目後再新增，會產生**重複的 id**，已改為遞增的 `nextId`。
- 功能越多（編輯、完成、篩選），程式碼會快速變成難以維護的 DOM 操作。

### Vue 版本

**優點**

- **資料驅動**：只要改 `data`，畫面自動更新，不用手動呼叫渲染函式。
- `v-model` 一行搞定輸入框雙向綁定，不需取得 DOM 元素。
- `v-for` + `@click` 讓模板直接表達畫面結構，可讀性高。
- Vue 使用虛擬 DOM 做差異更新，只更新有變動的部分。
- `{{ }}` 預設會跳脫 HTML，天然避免 XSS。
- 程式碼更短、更好維護，要擴充功能（編輯、完成、篩選）只需增加資料與 `computed`。
- 結構清楚（`data` / `methods` / 模板），方便團隊協作，也容易延伸到元件化、Vue CLI/Vite 專案。

**缺點**

- 需要載入 Vue 函式庫（CDN 或打包），有額外的檔案大小與相依性。
- 需要學習 Vue 的概念：響應式、指令（`v-model`、`v-for`、`@click`）、`this` 指向、生命週期。
- 對很小的頁面而言，有「殺雞用牛刀」的感覺。
- 抽象層較多，不熟時除錯較困難（例如響應式失效、`this` 在箭頭函式中的問題）。
- Vue 版同樣使用 `{ id, message }` 搭配 `:key="item.id"`，刪除中間項目時 key 也能正確對應。

### 結論與使用時機

| 情境 | 建議 |
| --- | --- |
| 單純的小頁面、只有一兩個互動 | 原生 JS 即可 |
| 需要頻繁更新畫面、有表單與清單 | 用 Vue，省下大量 DOM 操作 |
| 會持續擴充功能、多人協作 | 用 Vue（可再進階到元件化） |
| 想理解瀏覽器運作原理 | 先寫 JS，再轉 Vue，體會兩者差異 |

> 這個範例最大的收穫：**Vue 不是讓你「少寫幾行」，而是讓你「不用再管 DOM」**，把心力放在資料與邏輯上。
