[🏠 回到首頁](../../README.md)

# 06-Composition API 基礎

## 6-1 setup、reactive、ref 基礎

### 1. 什麼是 Composition API？

Composition API 是 Vue 3 提供的寫法：把「資料」與「操作資料的函式」用 JavaScript 函式組合在一起，而不是拆散到 `data`、`methods`、`computed` 等選項中（Options API）。

其他請參閱 [Composition API vs Option API](../../補充資料/03-Composition%20API%20vs%20Option%20API.md)

### 2. setup 與 `<script setup>`

| 寫法 | 說明 |
| --- | --- |
| `setup()` | CDN 寫法。必須 `return { ... }`，模板才能使用 |
| `<script setup>` | SFC 寫法（語法糖）。頂層宣告的變數、函式**自動**提供給 `<template>`，**不需要 `return`** |

```vue
<script setup>
import { ref } from "vue";

const message = ref("Hello Vue 3");
</script>

<template>
  <p>{{ message }}</p>
</template>
```

> 💡 本課程之後的範例一律使用 SFC 的 `<script setup>`。SFC 的結構請參閱 [SFC 補充資料](../../補充資料/05-SFC.md)。

### 3. 為什麼需要響應式變數？

**資料改變後畫面如果需要更新，才使用響應式變數（`reactive` 或 `ref`）；否則使用一般變數即可。**

一般變數雖然值會變，但 Vue 不知道它變了，所以畫面不會更新。

```vue
<script setup>
import { ref } from "vue";

// ❌ 一般變數：點擊後 console 看得到數字變大，但畫面不會變
let normalCount = 0;
const addNormal = () => {
  normalCount++;
  console.log("一般變數：", normalCount);
};

// ✅ 響應式變數：點擊後畫面自動更新
const refCount = ref(0);
const addRef = () => {
  refCount.value++;
};

// ✅ 不會變動的資料：用一般變數就好（不需要響應式）
const siteName = "Vue 3 筆記";
</script>

<template>
  <h1>{{ siteName }}</h1>

  <p>一般變數：{{ normalCount }}</p>
  <button @click="addNormal">一般變數 +1（畫面不更新）</button>

  <p>響應式變數：{{ refCount }}</p>
  <button @click="addRef">ref +1（畫面更新）</button>
</template>
```

> ⚠️ 不要把所有變數都包成 `ref`：不會改變、或改變後不需要更新畫面的資料，用一般變數即可，可減少不必要的監控成本。

### 4. reactive

`reactive()` 把**物件**轉成響應式物件，底層使用 ES6 的 **Proxy**，透過攔截 `getter`（讀取）與 `setter`（寫入）來追蹤資料變化。

| 特性 | 說明 |
| --- | --- |
| 底層原理 | `Proxy`，有 Getter 與 Setter |
| 可接受型別 | **只能處理 Object Type**（物件、陣列、`Map`、`Set`），不能放字串、數字、布林 |
| 存取方式 | 直接用屬性，**不需要** `.value` |
| 替換整個物件 | ❌ **不適合**：重新賦值會讓原本的響應式連結斷掉 |
| 可傳遞 ref | ✅ 可直接放入 `ref` 本身，存取時會**自動解開**（不用 `.value`） |

```vue
<script setup>
import { reactive, ref } from "vue";

// 1. 基本使用：直接以屬性存取
const user = reactive({
  name: "小明",
  age: 18,
});
const birthday = () => {
  user.age++; // 不需要 .value
};

// 2. 可直接傳遞 ref 本身：存取時自動解開
const score = ref(60);
const student = reactive({ name: "小華", score }); // 放入 ref
const addScore = () => {
  student.score += 10; // 不用寫 student.score.value
  console.log(score.value); // 同步改到原本的 ref
};

// 3. ❌ 不適合替換整個物件
let state = reactive({ list: [1, 2, 3] });
const wrongReplace = () => {
  // 換成全新物件後，模板綁定的仍是「舊的 Proxy」，畫面不會更新
  state = { list: [4, 5, 6] };
};
const rightReplace = () => {
  state.list = [4, 5, 6]; // ✅ 改裡面的屬性
};
</script>

<template>
  <p>{{ user.name }}：{{ user.age }} 歲</p>
  <button @click="birthday">生日快樂</button>

  <p>{{ student.name }}：{{ student.score }} 分</p>
  <button @click="addScore">加 10 分</button>

  <p>{{ state.list }}</p>
  <button @click="wrongReplace">整個替換（畫面不更新）</button>
  <button @click="rightReplace">替換屬性（畫面更新）</button>
</template>
```

### 5. ref

`ref()` 可以放**任何型別**，會回傳一個包裝物件 `{ value: ... }`。

| 特性 | 說明 |
| --- | --- |
| 可接受型別 | 字串、數字、布林、物件、陣列…都可以 |
| JS 中存取 | 必須使用 **`.value`** |
| 模板中存取 | **自動解開**，不需要 `.value` |
| 存放物件 | ref 也可以存放物件、陣列，物件內容會自動轉為 `reactive`（深層響應式） |
| 替換整個值 | ✅ 可以，`data.value = { ... }` |

```vue
<script setup>
import { ref } from "vue";

// 1. 基本型別
const count = ref(0);
const addCount = () => {
  count.value++; // JS 中要 .value
};

// 2. 存放物件：也要透過 .value，且內部屬性同樣是響應式
const user = ref({ name: "小明", age: 18 });
const birthday = () => {
  user.value.age++;
};

// 3. 可以整個替換（reactive 做不到）
const changeUser = () => {
  user.value = { name: "小華", age: 20 };
};

// 4. 存放陣列：常用於 API 回傳資料或清單
const todos = ref([]);
const addTodo = () => {
  todos.value.push({ id: Date.now(), text: "新的待辦事項" });
};
const clearTodos = () => {
  todos.value = []; // 整個清空
};
</script>

<template>
  <!-- 模板中不用寫 .value -->
  <p>{{ count }}</p>
  <button @click="addCount">+1</button>

  <p>{{ user.name }}：{{ user.age }} 歲</p>
  <button @click="birthday">生日快樂</button>
  <button @click="changeUser">整個替換</button>

  <ul>
    <li v-for="todo in todos" :key="todo.id">{{ todo.text }}</li>
  </ul>
  <button @click="addTodo">新增</button>
  <button @click="clearTodos">清空</button>
</template>
```

> ⚠️ **最常見的錯誤：** 在 JS 中忘記寫 `.value`。`count++` 是錯的（`count` 是物件，不是數字），要寫 `count.value++`。

---

## 6-2 如何選擇 ref 或 reactive

### 1. 快速比較

| 項目 | `ref` | `reactive` |
| --- | --- | --- |
| 可接受型別 | **任何型別** | 只能是物件型別 |
| JS 中存取 | 需要 `.value` | 直接使用屬性 |
| 模板中存取 | 自動解開 | 直接使用屬性 |
| 整個重新賦值 | ✅ 可以 | ❌ 會失去響應式 |
| 解構 | 解構出的 ref 仍有響應式 | ❌ 直接解構會失去響應式（需用 `toRefs`） |

### 2. 怎麼選？

| 情境 | 建議 |
| --- | --- |
| 字串、數字、布林等單一值 | `ref`（沒得選） |
| 會被整個覆蓋的資料（如 API 回傳） | `ref`：`data.value = res.data` |
| 陣列 | `ref`：方便整個替換、篩選後重新賦值 |
| 一組固定結構、彼此相關的欄位（如表單） | `reactive` 或 `ref` 皆可 |
| 不確定時 | **統一用 `ref`** |

> 初學者口訣：**不知道選哪個，就用 `ref`。** 團隊協作時，統一寫法比個人偏好更重要。

### 3. 常見陷阱：reactive 解構會失去響應式

```js
import { reactive, toRefs } from "vue";

const user = reactive({ name: "小明", age: 18 });

// ❌ 解構出來的 name 只是普通字串，改它不會影響畫面
let { name } = user;

// ✅ 用 toRefs 轉成 ref，保有響應式連結
const { name: nameRef, age } = toRefs(user);
nameRef.value = "小華"; // user.name 也會跟著變
```

其他請參閱 [補充資料：ref vs reactive](../../補充資料/04-ref%20vs%20reactive.md)

---

## 6-3 定義 Method

在 Composition API 中，**Method 就是一般的 JavaScript 函式**，不需要像 Options API 一樣放在 `methods` 裡，也不需要 `this`。

### 1. 定義與使用

在 `<script setup>` 內宣告函式，模板就可以直接使用（不需要 `return`）。

```vue
<script setup>
import { ref } from "vue";

const count = ref(0);

// 寫法一：箭頭函式（最常用）
const add = () => {
  count.value++;
};

// 寫法二：具名函式
function reduce() {
  count.value--;
}

// 帶參數的函式
const addBy = (num) => {
  count.value += num;
};

// 重置
const reset = () => {
  count.value = 0;
};
</script>

<template>
  <p>目前數字：{{ count }}</p>
  <button @click="add">+1</button>
  <button @click="reduce">-1</button>
  <button @click="addBy(10)">+10</button>
  <button @click="reset">重置</button>
</template>
```

### 2. 在模板中呼叫的寫法

| 寫法 | 範例 | 說明 |
| --- | --- | --- |
| 只傳函式名稱 | `@click="add"` | 點擊時才執行；Vue 會自動傳入事件物件 `event` |
| 帶括號傳參數 | `@click="addBy(10)"` | 點擊時才執行；可自訂傳入的參數 |
| 同時要事件物件 | `@click="log($event, 10)"` | 用 `$event` 取得原生事件物件 |
| 簡單的內嵌程式 | `@click="count++"` | 模板中直接寫，不需要 `.value` |

> ⚠️ **不要在模板中對 `@click` 寫成立即執行的呼叫：** `@click="add"` 才是「點擊時執行」。在 `{{ }}` 中寫 `{{ add() }}` 則是「每次渲染就執行」，會造成非預期的結果。

### 3. 與 Options API 的對照

| 項目 | Options API | Composition API |
| --- | --- | --- |
| 定義位置 | `methods: { add() {} }` | 直接宣告函式 `const add = () => {}` |
| 存取資料 | `this.count++` | `count.value++` |
| 是否需要 `this` | 需要 | **不需要** |
| 匯出給模板 | 自動 | `<script setup>` 自動；`setup()` 要 `return` |

> ⚠️ 箭頭函式在 Options API 的 `methods` 中會讓 `this` 失效；但在 Composition API 沒有 `this`，所以箭頭函式與一般函式都可以放心使用。

### 4. 綜合範例：待辦清單

```vue
<script setup>
import { ref } from "vue";

const text = ref("");
const todos = ref([]);

const addTodo = () => {
  if (text.value.trim() === "") return; // 空白不新增
  todos.value.push({ id: Date.now(), message: text.value });
  text.value = ""; // 清空輸入框
};

const removeTodo = (id) => {
  todos.value = todos.value.filter((item) => item.id !== id);
};
</script>

<template>
  <input v-model="text" @keyup.enter="addTodo" placeholder="輸入待辦事項" />
  <button @click="addTodo">新增</button>

  <ul>
    <li v-for="item in todos" :key="item.id">
      {{ item.message }}
      <button @click="removeTodo(item.id)">刪除</button>
    </li>
  </ul>
  <p v-if="todos.length === 0">目前沒有待辦事項</p>
</template>
```

---

## 6 章總結

1. 資料改變後**畫面要更新**才用 `ref` / `reactive`，否則用一般變數。
2. `reactive`：只限物件、使用 Proxy、不用 `.value`、**不能整個替換**、解構會失去響應式。
3. `ref`：任何型別、JS 中需要 `.value`、模板自動解開、**可整個替換**。
4. **不確定時一律用 `ref`**。
5. Method 就是一般函式，不需要 `this`，在 `<script setup>` 內宣告即可由模板使用。

---

## 📝 練習題

> 範圍：6-1 setup、reactive、ref 基礎、6-2 如何選擇 ref 或 reactive、6-3 定義 Method
> 建議先自己作答，再展開「答案與詳解」對照。

### 選擇題（共 10 題）

#### Q1. 使用 `<script setup>` 時，下列敘述何者正確？

- (A) 必須使用 `return` 才能讓模板使用變數
- (B) 頂層宣告的變數與函式可直接在 `<template>` 使用
- (C) 只能宣告 `ref`，不能宣告函式
- (D) 必須搭配 `export default` 才能運作

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`<script setup>` 是 Composition API 的語法糖，頂層的變數、函式會自動提供給模板，不需要 `return`。需要 `return` 的是 CDN 寫法的 `setup()`。

</details>

#### Q2. 下列哪一種資料**最不需要**使用 `ref` 或 `reactive`？

- (A) 點擊按鈕後會增加的計數器
- (B) 使用者輸入的搜尋關鍵字
- (C) 網站名稱常數 `const siteName = "我的網站"`，之後不會改變
- (D) 從 API 取得的商品清單

<details>
<summary>答案與詳解</summary>

**答案：(C)**

資料改變後畫面需要更新，才使用響應式變數。不會改變的常數用一般變數即可。

</details>

#### Q3. 關於 `reactive`，下列敘述何者**錯誤**？

- (A) 底層使用 Proxy
- (B) 只能處理物件型別
- (C) 在 JavaScript 中存取屬性需要加上 `.value`
- (D) 可以直接放入 `ref`，存取時會自動解開

<details>
<summary>答案與詳解</summary>

**答案：(C)**

`reactive` 直接用屬性存取，不需要 `.value`；需要 `.value` 的是 `ref`。

</details>

#### Q4. 下列程式碼執行後，會發生什麼事？

```js
const count = reactive(0);
```

- (A) 正常運作，`count` 為響應式數字
- (B) 無法使 `0` 成為響應式（`reactive` 不能放基本型別），且 Vue 會提出警告
- (C) `count` 會自動變成 `ref`
- (D) 自動轉為 `{ value: 0 }`

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`reactive` 只能處理物件型別，字串、數字、布林要使用 `ref`。

</details>

#### Q5. 在 `<script setup>` 的 JavaScript 中，要讓 `const count = ref(0)` 增加 1，正確寫法為何？

- (A) `count++`
- (B) `count.value++`
- (C) `count = count + 1`
- (D) `ref(count)++`

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`ref` 回傳 `{ value: ... }` 包裝物件，JS 中必須透過 `.value` 存取。`count++` 會把物件當數字運算，是常見錯誤。

</details>

#### Q6. 下列模板寫法，哪一個會**正確**顯示 `ref` 的內容？

```vue
<script setup>
const count = ref(0);
</script>
```

- (A) `{{ count.value }}`
- (B) `{{ count }}`
- (C) `{{ count() }}`
- (D) `{{ ref(count) }}`

<details>
<summary>答案與詳解</summary>

**答案：(B)**

模板中最上層的 `ref` 會**自動解開**，不需要 `.value`。寫成 `count.value` 在模板中會是 `undefined`（因為已解開成數字）。

</details>

#### Q7. 下列程式碼中，點擊按鈕後畫面為什麼**不會**更新？

```js
let state = reactive({ list: [1, 2, 3] });

const change = () => {
  state = { list: [4, 5, 6] };
};
```

- (A) `reactive` 不能存放陣列
- (B) 替換整個物件後，模板仍綁定舊的 Proxy，失去響應式連結
- (C) 必須使用 `const` 才能更新
- (D) 需要加上 `.value`

<details>
<summary>答案與詳解</summary>

**答案：(B)**

重新賦值會讓變數指向一個普通物件，原本的 Proxy 不再被使用。正確做法是改屬性（`state.list = [4, 5, 6]`），或改用 `ref`（`state.value = {...}`）。

</details>

#### Q8. 下列程式碼中，`name` 為什麼失去響應式？

```js
const user = reactive({ name: "小明", age: 18 });
let { name } = user;
name = "小華";
```

- (A) 解構出來的 `name` 只是普通字串，與 `user` 沒有連結
- (B) `reactive` 不能放字串
- (C) 變數名稱不能叫 `name`
- (D) 解構一定會報錯

<details>
<summary>答案與詳解</summary>

**答案：(A)**

解構取出的是**值的複製**，不再經過 Proxy。若要保有響應式，可使用 `toRefs(user)`。

</details>

#### Q9. 下列情境，哪一個**最適合**使用 `ref`？

- (A) 從 API 取得整包資料後，需要直接覆蓋原本的資料（`data.value = res.data`）
- (B) 絕對不會改變的常數
- (C) 要放入 `Symbol` 的單一資料
- (D) 以上皆非

<details>
<summary>答案與詳解</summary>

**答案：(A)**

`ref` 可整個替換，特別適合 API 回傳資料覆蓋。不會改變的常數用一般變數即可。

</details>

#### Q10. 關於 Composition API 中定義 Method，下列敘述何者正確？

- (A) 必須放在 `methods: {}` 中
- (B) 函式中存取資料要使用 `this.count`
- (C) 就是一般 JavaScript 函式，直接在 `<script setup>` 宣告即可，不需要 `this`
- (D) 只能使用 `function` 關鍵字，不能使用箭頭函式

<details>
<summary>答案與詳解</summary>

**答案：(C)**

Composition API 沒有 `this`，函式直接以閉包存取變數；箭頭函式與一般函式都可使用。

</details>

### 實作題（共 3 題）

#### P1. 計數器（ref + Method）

請完成一個計數器，需求如下：

1. 顯示目前數字，初始值為 `0`
2. 三個按鈕：`+1`、`-1`、`重置`，並以函式實作
3. 數字**不能小於 0**，小於 0 時 `-1` 不做事

<details>
<summary>參考解答</summary>

```vue
<script setup>
import { ref } from "vue";

const count = ref(0);

const add = () => {
  count.value++;
};

const reduce = () => {
  if (count.value > 0) count.value--;
};

const reset = () => {
  count.value = 0;
};
</script>

<template>
  <p>目前數字：{{ count }}</p>
  <button @click="add">+1</button>
  <button @click="reduce">-1</button>
  <button @click="reset">重置</button>
</template>
```

**重點：** JS 中使用 `.value`，模板中不用；用 `if` 判斷避免數字小於 0。

</details>

#### P2. 個人資料卡（reactive + Method）

請用 `reactive` 完成個人資料卡：

1. 資料：`name`（小明）、`age`（18）、`city`（台北）
2. 「年齡 +1」按鈕：讓年齡增加
3. 「搬家」按鈕：把城市改成 `高雄`
4. 「重置」按鈕：把資料還原為初始值（**注意：不能用 `user = {...}` 替換整個物件**）

<details>
<summary>參考解答</summary>

```vue
<script setup>
import { reactive } from "vue";

const initUser = { name: "小明", age: 18, city: "台北" };
const user = reactive({ ...initUser });

const addAge = () => {
  user.age++;
};

const move = () => {
  user.city = "高雄";
};

const reset = () => {
  Object.assign(user, initUser); // 逐一覆蓋屬性，保持響應式
};
</script>

<template>
  <p>姓名：{{ user.name }}</p>
  <p>年齡：{{ user.age }}</p>
  <p>城市：{{ user.city }}</p>
  <button @click="addAge">年齡 +1</button>
  <button @click="move">搬家</button>
  <button @click="reset">重置</button>
</template>
```

**重點：** `reactive` 不能整個替換，使用 `Object.assign(user, initUser)` 逐一覆蓋；初始值複製一份（`{ ...initUser }`），避免被修改。

</details>

#### P3. 待辦清單（ref + Method）

請用 `ref` 完成待辦清單：

1. 輸入框輸入文字，按「新增」加入清單；**空白不可新增**，新增後清空輸入框
2. 每筆資料有「刪除」按鈕
3. 清單為空時顯示「目前沒有待辦事項」
4. 「全部清除」按鈕：直接將清單設為空陣列

<details>
<summary>參考解答</summary>

```vue
<script setup>
import { ref } from "vue";

const text = ref("");
const todos = ref([]);

const addTodo = () => {
  if (text.value.trim() === "") return;
  todos.value.push({ id: Date.now(), message: text.value });
  text.value = "";
};

const removeTodo = (id) => {
  todos.value = todos.value.filter((item) => item.id !== id);
};

const clearAll = () => {
  todos.value = []; // ref 可整個替換
};
</script>

<template>
  <input v-model="text" @keyup.enter="addTodo" placeholder="輸入待辦事項" />
  <button @click="addTodo">新增</button>
  <button @click="clearAll">全部清除</button>

  <ul>
    <li v-for="item in todos" :key="item.id">
      {{ item.message }}
      <button @click="removeTodo(item.id)">刪除</button>
    </li>
  </ul>
  <p v-if="todos.length === 0">目前沒有待辦事項</p>
</template>
```

**重點：** `ref` 可整個替換（`filter` 結果、空陣列）；`trim()` 判斷空白；`id` 用 `Date.now()` 當唯一值。

</details>
