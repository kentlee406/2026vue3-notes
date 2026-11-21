[🏠 回到首頁](../../README.md)

# 04-Vue.js 模板指令與互動應用

## 4.1 模板語法與資料呈現

### 1. Vue 的兩個基本區塊

| 區塊 | 負責什麼 | 說明 |
| --- | --- | --- |
| `<template>` | **畫面（View）** | 用 HTML 搭配 Vue 的模板語法，描述畫面長什麼樣子 |
| `<script setup>` | **資料與功能** | 用 Composition API 宣告 `ref`、`reactive`、函式；宣告的變數可直接在 `<template>` 使用，不需要 `return` |

> 💡 目前的 CDN 寫法是 `setup() { ... return { ... } }`，需要自己 `return`；等到 Vite 專案（單一檔案元件 `.vue`）會改用 `<script setup>`，兩者概念相同，只是後者省略了 `return`。

**指令（Directive）：** 以 `v-` 開頭、寫在 HTML 標籤上的特殊屬性，用來告訴 Vue「這個元素要怎麼處理」。本章會學到 `v-html`、`v-bind`、`v-for`、`v-if`、`v-model`、`v-on`。

### 4.1.1 文字插值

使用**雙花括號** `{{ ... }}`（Mustache 語法），把資料顯示成**文字**。

`{{ }}` 內可以放**單一 JavaScript 表達式**（會有結果值的程式碼）：

| 可放入 | 範例 |
| --- | --- |
| JS 變數 | `{{ name }}` |
| 運算式 | `{{ price * 2 }}` |
| 樣板字面值 | `` {{ `NT$ ${price}` }} `` |
| 三元運算子 | `{{ stock > 0 ? '有庫存' : '已售完' }}` |
| 函式執行結果 | `{{ formatPrice(price) }}`、`{{ title.toUpperCase() }}` |

```html
<div id="app">
  <p>{{ name }}</p>
  <p>{{ price * 2 }}</p>
  <p>{{ `NT$ ${price}` }}</p>
  <p>{{ stock > 0 ? '有庫存' : '已售完' }}</p>
  <p>{{ title.toUpperCase() }}</p>
</div>
```

> ⚠️ 不能放**陳述式**：`{{ if (ok) { ... } }`、`{{ let a = 1 }}` 都會報錯。需要判斷請用三元運算子或 `v-if`。

### 4.1.2 v-html

`{{ }}` 會把內容當成**純文字**顯示（HTML 標籤會原樣印出）。若要把字串當成 **HTML** 解析，使用 `v-html`，作用相當於原生 JavaScript 的 `innerHTML`。

```html
<p>{{ rawHtml }}</p>          <!-- 顯示：<b>粗體</b>（原樣印出） -->
<p v-html="rawHtml"></p>      <!-- 顯示：粗體（真的變粗） -->
```

```js
const rawHtml = ref('<b>粗體</b>');
```

> ⚠️ **XSS（跨站腳本攻擊）風險：** 若 `v-html` 的內容來自使用者輸入或不可信的來源，攻擊者可塞入惡意 HTML / JavaScript。**只對可信任的內容使用 `v-html`，絕不要拿來渲染使用者輸入。**

### 4.1.3 v-bind

**文字插值不能用在標籤的屬性上**：

```html
<!-- ❌ 錯誤 -->
<p id={{ productID }}>編號</p>

<!-- ✅ 正確：v-bind -->
<p v-bind:id="productID">編號</p>

<!-- ✅ 簡寫：只留冒號 -->
<p :id="productID">編號</p>
```

`v-bind` 的雙引號內是 **JavaScript 表達式**，所以 `:id="productID"` 取的是變數；若要寫固定字串，要多包一層引號：`:id="'abc'"`（直接寫 `id="abc"` 更簡單）。

#### 1. 一般屬性與布林屬性

| 類型 | 例子 | Vue 的判定 |
| --- | --- | --- |
| 一般屬性 | `id`、`src`、`href`、`alt` | 值為 `null` 或 `undefined` 時，該屬性會被**移除**；其他值（包含 `0`、`false`）都會設為字串 |
| 布林屬性 | `disabled`、`checked`、`readonly`、`required` | 只要「**有出現**」就代表 `true` |

布林屬性的判定細節：

| 綁定的值 | 結果 |
| --- | --- |
| `true`、`''`（空字串）、非空字串 | 屬性**存在**（生效） |
| `false`、`null`、`undefined` | 屬性被**移除**（不生效） |

```html
<button :disabled="isLoading">送出</button>
<!-- isLoading 為 true  → <button disabled>   -->
<!-- isLoading 為 false → <button>（沒有 disabled 屬性） -->
```

> ⚠️ 原生 HTML 的 `<button disabled="false">` 仍然是**停用**（只要出現就生效），但在 Vue 中 `:disabled="false"` 會移除屬性，這是兩者的差異。

#### 2. 綁定整個物件

如果有多個屬性，可以把它們放進同一個物件，用不帶參數的 `v-bind` 一次綁定：

```html
<button v-bind="product">編號</button>
```

```js
const product = reactive({ id: 'p001', title: '商品', disabled: true });
// 等同於 <button id="p001" title="商品" disabled>
```

#### 3. 搭配 Class 與 Style 的綁定方式

`class` 與 `style` 用 `v-bind` 時，Vue 做了特別強化，可以傳入**物件**或**陣列**；而且可以和一般的 `class="..."` **同時存在**（會自動合併）。

**:class 物件語法：** `{ class名稱: 布林值 }`，值為 `true` 才加上該 class。

```html
<div class="box" :class="{ active: isActive, 'text-danger': hasError }"></div>
```

```js
const isActive = ref(true);
const hasError = ref(false);
// 結果：<div class="box active"></div>
```

> 💡 class 名稱含有連字號（如 `text-danger`）時，要用引號包起來。

**:class 陣列語法：** 陣列內的每一項都會成為 class，陣列中也可以放物件或三元運算子。

```html
<div :class="[activeClass, isBold ? 'bold' : '']"></div>
<div :class="[{ active: isActive }, errorClass]"></div>
```

**:style 物件語法：** 屬性名稱使用**駝峰式（camelCase）**，或用引號包起來的連字號寫法。

```html
<div :style="{ color: textColor, fontSize: fontSize + 'px' }"></div>
<div :style="{ 'background-color': bgColor }"></div>
<div :style="styleObject"></div>  <!-- 也可以直接綁定整個樣式物件 -->
```

| 寫法 | 適合情境 |
| --- | --- |
| `:class="{ a: 條件 }"` | 依**條件**切換某個 class（最常用） |
| `:class="[a, b]"` | 同時套用多個 class |
| `:style="{ ... }"` | 樣式值需要**動態計算**（顏色、寬度、字體大小） |

**實作：class-style.html**

```html
<div id="app">
  <div class="box" :class="{ active: isActive, error: hasError }">物件語法 :class</div>
  <div class="box" :class="[activeClass, { bold: isBold }]">陣列語法 :class</div>
  <div class="box" :style="{ color: textColor, fontSize: fontSize + 'px' }">物件語法 :style</div>

  <button type="button" @click="isActive = !isActive">切換 active</button>
  <button type="button" @click="fontSize += 2">字體 +2</button>
</div>
```

完整程式請見 [class-style.html](class-style.html)。

> ⚠️ 執行方式：請在 VS Code 對 `class-style.html` 按右鍵 → **Open with Live Server**。

### 4.1.4 v-for

使用 `v-for` 把**陣列、物件、數值**重複渲染成多個元素。

#### 1. 遍歷陣列：`(項目, 索引) in 陣列`

```html
<ul>
  <li v-for="(item, index) in products" :key="item.id">
    {{ index + 1 }}. {{ item.name }} - {{ item.price }}
  </li>
</ul>
```

```js
const products = ref([
  { id: 'p1', name: '蘋果', price: 30 },
  { id: 'p2', name: '香蕉', price: 20 },
]);
```

#### 2. 遍歷物件：`(值, 鍵, 索引) in 物件`

```html
<ul>
  <li v-for="(value, key, index) in user" :key="key">
    {{ index }}. {{ key }}：{{ value }}
  </li>
</ul>
```

> ⚠️ 注意順序：物件是 **值 → 鍵 → 索引**（先 value 再 key），和陣列的「項目、索引」不同。

#### 3. 遍歷數值：`n in 數字`

```html
<span v-for="n in 5" :key="n">{{ n }} </span>
<!-- 1 2 3 4 5（從 1 開始，不是 0） -->
```

#### 4. `:key` 為什麼重要？

`key` 是給 Vue 辨識「每個元素是誰」的唯一識別，資料新增、刪除、排序時，Vue 才能正確且有效率地更新畫面。

| 建議 | 說明 |
| --- | --- |
| 使用**唯一且穩定**的值 | 例如資料的 `id` |
| 避免用 `index` 當 key | 資料增刪、排序後 index 會改變，可能造成畫面或輸入框內容錯亂 |
| 不要用物件當 key | key 應為字串或數字 |

> 💡 `v-for` 搭配 `v-if` 時，**不建議寫在同一個元素上**（`v-if` 優先權較高，無法取得 `v-for` 的變數）。可以改成先用 `computed` 過濾資料，或在外層包一層 `<template v-for>`。

## 4.2 條件與表單

### 4.2.1 v-if

依條件決定元素「要不要存在」於畫面中。

```html
<p v-if="score >= 90">優秀</p>
<p v-else-if="score >= 60">及格</p>
<p v-else>不及格</p>
```

- `v-if`、`v-else-if`、`v-else` 必須**緊鄰**在一起，中間不能插入其他元素。
- 條件為 `false` 時，元素會從 DOM 中**移除**（不是隱藏）。
- 要一次控制多個元素，可包在 `<template v-if="...">` 中，`<template>` 本身不會產生在 DOM 裡。

**v-if 與 v-show 的差異**

| 項目 | `v-if` | `v-show` |
| --- | --- | --- |
| 條件為 false 時 | 元素從 DOM **移除** | 元素仍在 DOM，只是加上 `display: none` |
| 切換成本 | 較高（要重新建立／銷毀元素） | 較低（只改 CSS） |
| 初次渲染成本 | 條件為 false 時不渲染，較低 | 一律渲染，較高 |
| 支援 `v-else` | ✅ | ❌ |
| 適合情境 | 條件**很少改變** | 需要**頻繁切換**顯示／隱藏 |

### 4.2.2 v-model

在表單元素上建立**雙向綁定**（資料 ⇄ 畫面，如 3.1 所述）。`v-model` 會依元素類型自動選擇對應的屬性與事件，不需要自己處理。

| 表單元素 | 綁定的資料型態 | 範例 |
| --- | --- | --- |
| `<input type="text">`、`<textarea>` | 字串 | `<input v-model="name">` |
| 單一 `<input type="checkbox">` | 布林值 | `<input type="checkbox" v-model="agree">` |
| 多個 `<input type="checkbox">` | **陣列**（存放被勾選的 `value`） | `v-model="fruits"` |
| `<input type="radio">` | 字串（被選取的 `value`） | `v-model="gender"` |
| `<select>` | 字串（被選取的 `value`） | `<select v-model="city">` |

```html
<!-- 多個核取方塊：資料要宣告成陣列 -->
<label><input type="checkbox" value="蘋果" v-model="fruits"> 蘋果</label>
<label><input type="checkbox" value="香蕉" v-model="fruits"> 香蕉</label>
<p>已選：{{ fruits }}</p>

<!-- 單選按鈕：同一組綁同一個資料 -->
<label><input type="radio" value="男" v-model="gender"> 男</label>
<label><input type="radio" value="女" v-model="gender"> 女</label>

<!-- 下拉選單 -->
<select v-model="city">
  <option disabled value="">請選擇</option>
  <option value="taipei">台北</option>
  <option value="taichung">台中</option>
</select>
```

```js
const fruits = ref([]);   // 多選 → 陣列
const gender = ref('');
const city = ref('');
```

**v-model 修飾符**

| 修飾符 | 作用 | 範例 |
| --- | --- | --- |
| `.lazy` | 改成在 `change` 事件（失去焦點／按 Enter）才同步，而非每次輸入都同步 | `v-model.lazy="msg"` |
| `.number` | 把輸入的字串**自動轉成數字** | `v-model.number="age"` |
| `.trim` | 自動去除前後空白 | `v-model.trim="name"` |

> ⚠️ `<input type="text">` 取到的值**一律是字串**，輸入 `10` 得到的是 `'10'`。要做數學運算時，請加上 `.number`，否則 `'10' + '5'` 會變成 `'105'`。

> 💡 `v-model` 會忽略表單元素上原本的 `value`、`checked`、`selected` 屬性，初始值一律以 JavaScript 的資料為準。

## 4.3 事件處理 v-on

使用 `v-on`（簡寫 `@`）監聽 DOM 事件，事件發生時執行指定的程式。

```html
<button v-on:click="add">+1</button>   <!-- 完整寫法 -->
<button @click="add">+1</button>       <!-- 簡寫（常用） -->
```

### 1. 三種常見寫法

```html
<!-- 1. 直接寫簡單的表達式 -->
<button @click="count++">+1</button>

<!-- 2. 呼叫函式（不帶括號：自動收到事件物件 event） -->
<button @click="add">+1</button>

<!-- 3. 呼叫函式並傳參數；需要事件物件時用 $event -->
<button @click="remove(item.id)">刪除</button>
<button @click="remove(item.id, $event)">刪除</button>
```

> ⚠️ `@click="add"` 與 `@click="add()"` 都是「點擊時」才執行。真正的錯誤是在 `<script>` 中寫 `btn.onclick = add()`，那會在**指定當下就立即執行**。

### 2. 事件修飾符

寫在事件名稱後面，以 `.` 連接，可串接多個（**順序有意義**）。

| 修飾符 | 作用 | 常見用途 |
| --- | --- | --- |
| `.prevent` | 呼叫 `event.preventDefault()`，阻止預設行為 | `@submit.prevent`：表單送出時不重新整理頁面 |
| `.stop` | 呼叫 `event.stopPropagation()`，阻止事件往上冒泡 | 子元素點擊不觸發父元素事件 |
| `.self` | 只有事件是**自己本身**觸發時才執行（子元素觸發不算） | 點擊彈窗背景才關閉 |
| `.capture` | 改用「捕獲」階段監聽 | 少用 |
| `.once` | 事件**只會觸發一次** | 一次性的操作 |
| `.passive` | 告訴瀏覽器不會呼叫 `preventDefault()`，提升捲動效能 | `@scroll.passive`、`@touchmove.passive` |

```html
<form @submit.prevent="send">...</form>

<div @click="parentClick">
  <button @click.stop="childClick">按我不會觸發外層</button>
</div>
```

### 3. 鍵盤修飾符

監聽鍵盤事件（`@keyup`、`@keydown`）時，可指定要回應的按鍵。

| 修飾符 | 按鍵 |
| --- | --- |
| `.enter` | Enter |
| `.tab` | Tab |
| `.delete` | Delete 與 Backspace |
| `.esc` | Esc |
| `.space` | 空白鍵 |
| `.up` `.down` `.left` `.right` | 方向鍵 |

```html
<input type="text" v-model="keyword" @keyup.enter="search">
<input type="text" @keyup.esc="clear">
```

**系統修飾鍵：** `.ctrl`、`.alt`、`.shift`、`.meta`（Windows 鍵／Mac 的 Command）。

```html
<input @keyup.ctrl.enter="send">   <!-- 按住 Ctrl 再按 Enter 才觸發 -->
<button @click.ctrl.exact="a">只有按著 Ctrl（沒按其他系統鍵）時才觸發</button>
```

`.exact`：精準控制，不允許有其他系統修飾鍵一起按下。

### 4. 滑鼠修飾符

| 修飾符 | 說明 |
| --- | --- |
| `.left` | 滑鼠左鍵 |
| `.right` | 滑鼠右鍵（常搭配 `@contextmenu.prevent` 取代預設選單） |
| `.middle` | 滑鼠中鍵 |

```html
<div @mousedown.left="a" @mousedown.right="b" @mousedown.middle="c">試試看三種按鍵</div>
<div @contextmenu.prevent="showMenu">在這裡按右鍵</div>
```

### 5. 實作：簡單的待辦輸入

```html
<div id="app">
  <input type="text" v-model.trim="newItem" @keyup.enter="addItem" placeholder="輸入後按 Enter">
  <button type="button" @click="addItem">新增</button>
  <ul>
    <li v-for="(item, index) in list" :key="item.id">
      {{ item.text }}
      <button type="button" @click="removeItem(index)">刪除</button>
    </li>
  </ul>
</div>

<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  const { createApp, ref } = Vue;

  createApp({
    setup() {
      const newItem = ref('');
      const list = ref([]);

      const addItem = () => {
        if (!newItem.value) return;
        list.value.push({ id: Date.now(), text: newItem.value });
        newItem.value = '';
      };

      const removeItem = (index) => {
        list.value.splice(index, 1);
      };

      return { newItem, list, addItem, removeItem };
    }
  }).mount('#app');
</script>
```

### 6. 常見錯誤

| 錯誤 | 原因 | 解法 |
| --- | --- | --- |
| 屬性沒有綁定成功，畫面出現 `{{ id }}` 原文 | 在屬性中使用了 `{{ }}` | 改用 `:id="id"` |
| 畫面出現 `v-for` 警告：Elements in iteration expect to have 'v-bind:key' | 沒有加 `:key` | 加上 `:key="item.id"` |
| 輸入數字後相加變成字串相接（`'1' + '2'` = `'12'`） | `v-model` 取得的是字串 | 使用 `v-model.number` |
| 表單一送出頁面就重新整理 | 沒有阻止預設行為 | 使用 `@submit.prevent` |
| 多選核取方塊只得到 `true` / `false` | 綁定的資料不是陣列 | 資料改為 `ref([])` |
| `v-else` 沒有作用、出現警告 | `v-if` 與 `v-else` 中間夾了其他元素 | 讓它們**緊鄰**在一起 |

---

## 本章重點

1. **文字插值** `{{ }}` 只能放單一表達式；`v-html` 可渲染 HTML，但有 **XSS** 風險，勿用於使用者輸入。
2. **v-bind**（`:`）用於屬性；布林屬性值為 `false`、`null`、`undefined` 時會被移除；`:class`、`:style` 可傳入物件或陣列。
3. **v-for** 可遍歷陣列、物件（值→鍵→索引）、數值（從 1 開始），務必加上唯一的 `:key`，不要用 `index`。
4. **v-if** 會增刪 DOM、可搭配 `v-else`；**v-show** 只切換 `display`，適合頻繁切換。
5. **v-model** 是雙向綁定：多選核取方塊用陣列，修飾符有 `.lazy`、`.number`、`.trim`。
6. **v-on**（`@`）綁定事件，修飾符有 `.prevent`、`.stop`、`.self`、`.once`；鍵盤有 `.enter`、`.esc`；滑鼠有 `.left`、`.right`、`.middle`。

---

## 📝 練習題

> 範圍：4.1 模板語法與資料呈現、4.2 條件與表單、4.3 事件處理 v-on
> 建議做法：先自己作答，再展開「答案與詳解」對照。

---

### 一、選擇題（共 10 題）

#### Q1. 下列哪一個 `{{ }}` 的寫法**無法**正常運作？

- (A) `{{ price * 2 }}`
- (B) `{{ stock > 0 ? '有庫存' : '已售完' }}`
- (C) `{{ title.toUpperCase() }}`
- (D) `{{ if (stock > 0) { '有庫存' } }}`

<details>
<summary>答案與詳解</summary>

**答案：(D)**

`{{ }}` 內只能放**單一表達式**（有結果值的程式碼），不能放 `if` 這類陳述式。需要判斷時可改用三元運算子（B）或 `v-if`。

</details>

#### Q2. 已知 `const msg = ref('<b>Hello</b>')`，下列敘述何者正確？

- (A) `<p>{{ msg }}</p>` 會顯示粗體的 Hello
- (B) `<p v-html="msg"></p>` 會顯示粗體的 Hello
- (C) `<p v-html="msg"></p>` 沒有任何安全疑慮，可放心渲染使用者輸入
- (D) `<p>{{ msg }}</p>` 會產生 XSS 風險

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`{{ }}` 把內容當純文字，會原樣印出 `<b>Hello</b>`，且較安全；`v-html` 相當於 `innerHTML`，會解析標籤顯示粗體，但若內容來自使用者，會有 **XSS** 風險，所以 (C) 錯。

</details>

#### Q3. 若要把變數 `productID` 綁定到 `<p>` 的 `id` 屬性，下列哪一個寫法**正確**？

- (A) `<p id="{{ productID }}">編號</p>`
- (B) `<p id=productID>編號</p>`
- (C) `<p :id="productID">編號</p>`
- (D) `<p v-bind:id="{{ productID }}">編號</p>`

<details>
<summary>答案與詳解</summary>

**答案：(C)**

屬性要用 `v-bind`（簡寫 `:`），引號內直接寫變數名稱，**不能再使用** `{{ }}`。(A)(D) 錯在使用雙花括號，(B) 則是把 `productID` 當成固定文字。

</details>

#### Q4. 已知 `const isLoading = ref(false)`，下列程式渲染後的結果為何？

```html
<button :disabled="isLoading">送出</button>
```

- (A) `<button disabled="false">送出</button>`
- (B) `<button disabled>送出</button>`
- (C) `<button>送出</button>`（沒有 disabled 屬性，按鈕可以點）
- (D) 會報錯，因為布林屬性不能使用 `v-bind`

<details>
<summary>答案與詳解</summary>

**答案：(C)**

布林屬性的值為 `false`、`null`、`undefined` 時，Vue 會把該屬性**移除**，按鈕可正常點擊；值為 `true`（或空字串）時才會加上 `disabled`。

</details>

#### Q5. 已知 `isActive = true`、`hasError = false`，下列元素最後的 `class` 為何？

```html
<div class="box" :class="{ active: isActive, 'text-danger': hasError }"></div>
```

- (A) `box`
- (B) `box active`
- (C) `box active text-danger`
- (D) `active`

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`:class` 的物件語法中，值為 `true` 的 class 才會被加入，並且會與靜態的 `class="box"` **合併**，所以是 `box active`。`hasError` 為 `false`，`text-danger` 不會出現。

</details>

#### Q6. 下列程式渲染後，畫面會顯示什麼？

```html
<span v-for="n in 3" :key="n">{{ n }} </span>
```

- (A) `0 1 2`
- (B) `1 2 3`
- (C) `1 2 3 4`
- (D) 會報錯，`v-for` 不能用數值

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`v-for` 可以遍歷數值，且是從 **1** 開始計算（不是 0），所以 `n in 3` 依序是 1、2、3。

</details>

#### Q7. 關於 `v-if` 與 `v-show`，何者**正確**？

- (A) `v-show` 為 `false` 時，元素會從 DOM 中移除
- (B) `v-if` 為 `false` 時，元素只是加上 `display: none`
- (C) 需要頻繁切換顯示與隱藏時，較適合使用 `v-show`
- (D) `v-show` 可以搭配 `v-else`

<details>
<summary>答案與詳解</summary>

**答案：(C)**

`v-if` 會真的新增／移除 DOM（切換成本較高）；`v-show` 只切換 CSS 的 `display`（切換成本較低），所以頻繁切換用 `v-show`。(A)(B) 剛好說反了；`v-show` **不支援** `v-else`，(D) 錯。

</details>

#### Q8. 使用者在輸入框輸入 `10`，下列哪個寫法能讓 `age` 的值是**數字** `10` 而不是字串？

- (A) `<input v-model="age">`
- (B) `<input v-model.number="age">`
- (C) `<input v-model.lazy="age">`
- (D) `<input v-model.trim="age">`

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`input` 取到的值預設一律是字串。`.number` 會自動把輸入值轉成數字；`.lazy` 是改成 `change` 事件才同步；`.trim` 是去除前後空白。

</details>

#### Q9. 下列程式碼，點擊「按鈕」時，會執行哪些函式？

```html
<div @click="parentClick">
  <button @click.stop="childClick">按鈕</button>
</div>
```

- (A) 只有 `childClick`
- (B) 只有 `parentClick`
- (C) 先 `childClick`，再 `parentClick`
- (D) 先 `parentClick`，再 `childClick`

<details>
<summary>答案與詳解</summary>

**答案：(A)**

`.stop` 等同於 `event.stopPropagation()`，會**阻止事件往上冒泡**，所以外層 `div` 的 `parentClick` 不會被觸發。若沒有 `.stop`，答案才會是 (C)。

</details>

#### Q10. 希望使用者在輸入框中按下 **Enter** 時執行 `search`，且表單送出時不要重新整理頁面，下列哪一組寫法最合適？

- (A) `<input @keyup.enter="search">` 與 `<form @submit.prevent="send">`
- (B) `<input @keyup.space="search">` 與 `<form @submit.stop="send">`
- (C) `<input @click.enter="search">` 與 `<form @submit.once="send">`
- (D) `<input @keyup.esc="search">` 與 `<form @submit.self="send">`

<details>
<summary>答案與詳解</summary>

**答案：(A)**

`.enter` 是對應 Enter 鍵的鍵盤修飾符（搭配 `keyup`／`keydown`）；`.prevent` 會阻止表單預設的送出重整行為。`.space` 是空白鍵、`.esc` 是 Esc 鍵；`.stop` 是阻止冒泡，不是阻止預設行為。

</details>

---

### 二、實作題（共 2 題）

#### 實作 1：商品清單（v-for、v-bind、v-if）

請使用 CDN 的 Vue 3，完成下列需求：

1. 準備一個商品陣列，每筆含 `id`、`name`、`price`、`stock`（庫存），至少 4 筆，其中 **1 筆庫存為 0**。
2. 用 `v-for` 列出商品，顯示「編號. 商品名稱 - NT$ 價格（庫存 N）」，並加上正確的 `:key`。
3. 庫存為 0 的商品，套用 `sold-out` class（文字變灰、加刪除線），且「加入購物車」按鈕要 **disabled**（使用 `:class` 與 `:disabled`）。
4. 價格 **大於等於 1000** 的商品，價格文字用 `:style` 顯示紅色。
5. 點「加入購物車」按鈕，該商品庫存減 1。
6. 加上「清空商品」按鈕，清空後使用 `v-if` / `v-else` 顯示「目前沒有商品」。

<details>
<summary>答案與詳解</summary>

```html
<!doctype html>
<html lang="zh-Hant">
  <head>
    <meta charset="UTF-8" />
    <title>商品清單</title>
    <style>
      .sold-out {
        color: gray;
        text-decoration: line-through;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <h1>商品清單</h1>
      <button type="button" @click="clearProducts">清空商品</button>

      <p v-if="products.length === 0">目前沒有商品</p>
      <ul v-else>
        <li v-for="(item, index) in products" :key="item.id" :class="{ 'sold-out': item.stock === 0 }">
          {{ index + 1 }}. {{ item.name }} -
          <span :style="{ color: item.price >= 1000 ? 'red' : 'inherit' }">NT$ {{ item.price }}</span>
          （庫存 {{ item.stock }}）
          <button type="button" :disabled="item.stock === 0" @click="addToCart(item)">加入購物車</button>
        </li>
      </ul>
    </div>

    <script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
    <script>
      const { createApp, ref } = Vue;

      createApp({
        setup() {
          const products = ref([
            { id: 'p1', name: '蘋果', price: 30, stock: 5 },
            { id: 'p2', name: '筆記型電腦', price: 25000, stock: 2 },
            { id: 'p3', name: '耳機', price: 1200, stock: 0 },
            { id: 'p4', name: '滑鼠', price: 590, stock: 10 },
          ]);

          const addToCart = (item) => {
            if (item.stock > 0) {
              item.stock--;
            }
          };

          const clearProducts = () => {
            products.value = [];
          };

          return { products, addToCart, clearProducts };
        },
      }).mount('#app');
    </script>
  </body>
</html>
```

**詳解：**

- `:key` 使用資料唯一的 `id`，而不是 `index`。
- `:class="{ 'sold-out': item.stock === 0 }"`：條件成立才加上 class；class 名稱含連字號要用引號包起來。
- `:disabled="item.stock === 0"`：值為 `false` 時 Vue 會移除 `disabled` 屬性，按鈕才能點。
- `:style` 用物件語法搭配三元運算子，依價格動態決定顏色。
- `products` 是 `ref` 包住的陣列，陣列內的物件也是響應式的，所以 `item.stock--` 畫面會自動更新；在 `<script>` 中要替換整個陣列需寫 `products.value = []`。
- `v-if` 與 `v-else` 必須緊鄰：`<p v-if>` 之後立刻接 `<ul v-else>`。

</details>

#### 實作 2：待辦清單（v-model、v-on、修飾符）

請完成下列需求：

1. 輸入框使用 `v-model.trim`，輸入內容後，**按 Enter** 或 **點「新增」按鈕** 都可新增待辦（內容為空不新增，新增後清空輸入框）。
2. 每筆待辦前有一個 **核取方塊**（`v-model` 綁定 `done`），勾選後文字加上刪除線（使用 `:class`）。
3. 每筆待辦後有「刪除」按鈕，點擊後刪除該筆。
4. 畫面下方顯示「未完成：N 項」，N 為尚未勾選的數量（提示：可用 `filter` 計算）。
5. 加上一個選單 `<select v-model="filter">`，選項為「全部／未完成／已完成」，依選擇顯示對應的待辦。
6. 整個輸入區包在 `<form>` 中，使用 `@submit.prevent` 處理新增，避免頁面重新整理。

<details>
<summary>答案與詳解</summary>

```html
<!doctype html>
<html lang="zh-Hant">
  <head>
    <meta charset="UTF-8" />
    <title>待辦清單</title>
    <style>
      .done {
        color: gray;
        text-decoration: line-through;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <h1>待辦清單</h1>

      <form @submit.prevent="addTodo">
        <input type="text" v-model.trim="newTodo" placeholder="輸入待辦事項後按 Enter" />
        <button type="submit">新增</button>
      </form>

      <label>
        顯示：
        <select v-model="filter">
          <option value="all">全部</option>
          <option value="active">未完成</option>
          <option value="completed">已完成</option>
        </select>
      </label>

      <ul>
        <li v-for="todo in filteredTodos" :key="todo.id">
          <input type="checkbox" v-model="todo.done" />
          <span :class="{ done: todo.done }">{{ todo.text }}</span>
          <button type="button" @click="removeTodo(todo.id)">刪除</button>
        </li>
      </ul>

      <p v-if="filteredTodos.length === 0">沒有待辦事項</p>
      <p>未完成：{{ todos.filter((t) => !t.done).length }} 項</p>
    </div>

    <script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
    <script>
      const { createApp, ref, computed } = Vue;

      createApp({
        setup() {
          const newTodo = ref('');
          const filter = ref('all');
          const todos = ref([]);

          const addTodo = () => {
            if (!newTodo.value) return;
            todos.value.push({ id: Date.now(), text: newTodo.value, done: false });
            newTodo.value = '';
          };

          const removeTodo = (id) => {
            todos.value = todos.value.filter((todo) => todo.id !== id);
          };

          // computed：依 filter 的值算出要顯示的清單，資料變動時自動重新計算
          const filteredTodos = computed(() => {
            if (filter.value === 'active') return todos.value.filter((t) => !t.done);
            if (filter.value === 'completed') return todos.value.filter((t) => t.done);
            return todos.value;
          });

          return { newTodo, filter, todos, filteredTodos, addTodo, removeTodo };
        },
      }).mount('#app');
    </script>
  </body>
</html>
```

**詳解：**

- `<form @submit.prevent="addTodo">`：按 Enter 或點 `type="submit"` 的按鈕都會觸發 `submit`，`.prevent` 阻止頁面重新整理，所以不需要另外寫 `@keyup.enter`。
- `v-model.trim`：自動去除前後空白，因此只輸入空白也會被視為空字串而不新增。
- `v-model="todo.done"`：核取方塊綁定單一布林值；因為 `todos` 是響應式資料，勾選後 `:class="{ done: todo.done }"` 會自動更新。
- 刪除使用 `filter` 產生**不含該 id 的新陣列**，並以 `id`（而非 `index`）辨識，避免在「過濾顯示」時刪錯項目。
- `filteredTodos` 使用 `computed` 依 `filter` 回傳不同的陣列，`v-for` 遍歷的是過濾後的結果；`computed` 會在後續章節詳細介紹，此處先理解「由原有資料算出新資料」的用途即可。

</details>
