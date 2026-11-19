[🏠 回到首頁](../README.md)

# 04-ref vs reactive

## 一、ref 與 reactive 的比較

### 1-1 先搞懂：為什麼需要它們？

在 Vue 3 的 Composition API 中，一般變數改變時，**畫面不會跟著更新**。必須把資料宣告成「**響應式（Reactive）資料**」，Vue 才會監測它的變化並自動更新畫面。

```js
// ❌ 一般變數：count++ 後畫面不會變
let count = 0;

// ✅ 響應式：count.value++ 後畫面自動更新
const count = ref(0);
```

> 上面是「二選一」的對照寫法，不能放在同一份程式裡（同名變數不能重複宣告）。

Vue 3 提供兩個宣告響應式資料的函式：

- `ref()`：什麼型別都能放
- `reactive()`：只能放「物件」（含陣列）

> 一句話：**`ref` 是「把值裝進盒子」，`reactive` 是「把整個物件變成會被監看的物件」。**

### 1-2 基本概念

| 項目             | `ref`                                                        | `reactive`                                               |
| ---------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| 可接受的型別     | **任何型別**：字串、數字、布林、物件、陣列                   | **只能是物件型別**：物件、陣列（不能放字串、數字、布林） |
| 回傳結果         | 一個包裝物件 `{ value: ... }`                                | 原物件的 Proxy（代理）                                   |
| JS 中存取        | 必須使用 **`.value`**                                        | **直接**使用屬性，不需要 `.value`                        |
| 模板中存取       | 自動解開，**不用** `.value`                                  | 直接使用                                                 |
| 重新賦值整個資料 | ✅ 可以：`data.value = { ... }`                              | ❌ 不行（會失去響應式）                                  |
| 解構             | 解構出的 `ref` 本身仍有響應式                                | ❌ 直接解構會失去響應式（需用 `toRefs`）                 |
| 底層原理         | 基本型別用 `getter / setter` 包裝；物件則內部轉成 `reactive` | 使用 ES6 `Proxy`                                         |

### 1-3 程式碼對照

#### ① 基本型別：只有 `ref` 能用

```js
import { ref, reactive } from "vue";

const count = ref(0);
count.value++; // JS 中要 .value
console.log(count.value); // 1

const num = reactive(0); // ❌ 無效！reactive 不能放基本型別（會警告，且不具響應式）
```

#### ② 物件：兩者都可以，差在 `.value`

```js
// ===== ref =====
const user = ref({ name: "小明", age: 18 });
user.value.age++; // 要 .value
console.log(user.value.name);

// ===== reactive =====
const user2 = reactive({ name: "小明", age: 18 });
user2.age++; // 不用 .value，直接點屬性
console.log(user2.name);
```

#### ③ 模板中的寫法（兩者都不用 `.value`）

```vue
<script setup>
import { ref, reactive } from "vue";
const count = ref(0);
const user = reactive({ name: "小明", age: 18 });
</script>

<template>
  <p>{{ count }}</p>
  <!-- ref：模板自動解開，不用 .value -->
  <p>{{ user.name }}</p>
  <button @click="count++">加一</button>
</template>
```

#### ④ 陷阱一：`reactive` 不能整個換掉

```js
let state = reactive({ list: [1, 2, 3] });

// ❌ 整個換掉：原本的響應式連結斷了，畫面不更新
state = { list: [4, 5, 6] };

// ✅ 只改裡面的屬性
state.list = [4, 5, 6];
```

```js
const state = ref({ list: [1, 2, 3] });

// ✅ ref 可以整個換掉（因為你換的是 .value）
state.value = { list: [4, 5, 6] };
```

> 實務上最常遇到的情境：**API 回傳整包資料要直接覆蓋**。用 `ref` 直接 `data.value = res.data` 最方便；用 `reactive` 就得一個一個屬性賦值，或用 `Object.assign`。

#### ⑤ 陷阱二：`reactive` 解構會失去響應式

```js
import { reactive, toRefs } from "vue";

const user = reactive({ name: "小明", age: 18 });

// ❌ 解構出來的 name 只是普通字串，改它不會影響畫面
let { name } = user;

// ✅ 用 toRefs 轉成 ref，保有響應式連結
const { name: nameRef, age } = toRefs(user);
nameRef.value = "小華"; // user.name 也會跟著變
```

#### ⑥ 陷阱三：忘記寫 `.value`

```js
const count = ref(0);

count++; // ❌ 錯誤：count 是物件，不是數字
count.value++; // ✅ 正確
```

#### ⑦ 陷阱四：模板只會自動解開「最上層」的 `ref`

```vue
<script setup>
import { ref } from "vue";
const count = ref(0);
const obj = { count: ref(0) };
</script>

<template>
  <p>{{ count }}</p>
  <!-- ✅ 最上層的 ref：自動解開 -->
  <p>{{ obj.count }}</p>
  <!-- ❌ 包在一般物件裡的 ref：不會解開，會顯示成 ref 物件 -->
  <p>{{ obj.count.value }}</p>
  <!-- ✅ 要自己寫 .value -->
</template>
```

> 若想避免這個問題，可以把 `obj` 改用 `reactive` 包起來，或直接把 `count` 放在最上層。

#### ⑧ 補充：`ref` 放進 `reactive` 會自動解開

```js
const count = ref(0);
const state = reactive({ count });

console.log(state.count); // 0（不需要 .value）
state.count++; // 同步改到 count.value
console.log(count.value); // 1
```

#### ⑨ 補充：`watch` 監看的寫法不同

```js
const count = ref(0);
const state = reactive({ name: "小明" });

// ref：直接傳入即可
watch(count, (newVal) => console.log(newVal));

// reactive 的屬性：必須用 getter 函式（不能直接寫 state.name）
watch(
  () => state.name,
  (newVal) => console.log(newVal),
);
```

### 1-4 搭配上一課的 todolist 範例

```js
// 用 ref：每個狀態各一個
const todos = ref([]);
const text = ref("");
todos.value.push({ id: 1, message: text.value });

// 用 reactive：把相關狀態合成一個物件
const state = reactive({ todos: [], text: "" });
state.todos.push({ id: 1, message: state.text });
```

### 1-5 優缺點整理

| 項目 | `ref`                                                | `reactive`                                 |
| ---- | ---------------------------------------------------- | ------------------------------------------ |
| 優點 | 型別不限制；可整個重新賦值；解構與傳遞不易失去響應式 | 寫法直覺，不用 `.value`，像一般物件        |
| 缺點 | JS 中要一直寫 `.value`，容易漏掉                     | 限物件型別；不能整個換掉；解構會失去響應式 |

### 1-6 怎麼選？

| 情境                                       | 建議                                  |
| ------------------------------------------ | ------------------------------------- |
| 字串、數字、布林等**單一值**               | `ref`（沒得選）                       |
| 會被**整個覆蓋**的資料（如 API 回傳）      | `ref`                                 |
| 陣列                                       | `ref`（方便整個替換、篩選後重新賦值） |
| 一組**固定結構、彼此相關**的欄位（如表單） | `reactive` 或 `ref` 皆可              |
| 不確定時                                   | **統一用 `ref`**                      |

> 初學者口訣：**不知道選哪個，就用 `ref`。**

### 1-7 一張圖看懂

```
ref(0)                         reactive({ a: 1 })
┌──────────────┐               ┌──────────────┐
│ { value: 0 } │               │ Proxy        │
└──────────────┘               │ { a: 1 }     │
JS 讀寫： x.value               └──────────────┘
模板讀寫： x（自動解開）         JS / 模板讀寫： x.a
```

---

## 二、影片重點摘錄

影片：[空檔的交流，尤雨溪淺談 ref 跟 reactive 的看法](https://www.youtube.com/watch?v=e8Wlv4AGJjk)

1. **`reactive` 的使用感受接近 Vue 2**：Vue 2 的 `data()` 是一包物件，用 `this.xxx` 直接存取，不需要 `.value`；`reactive` 的 `state.xxx` 寫法與它很像，Vue 2 轉過來的人比較容易上手。

2. **但 `reactive` 在後期重構時比較吃虧**：把資料拆出去（例如抽成 composable、傳給函式、解構）時，會失去響應式，必須改用 `toRefs` 補救（見 1-3 ⑤）；而且只能存放物件型別。

3. **`ref` 與 `reactive` 有一部分是個人主觀偏好**：兩者都能達成目標，沒有絕對的對錯。

4. **團隊協作時要統一**：不論選哪一種，團隊內應有共識、寫法一致，比個人偏好更重要。官方文件則是建議以 `ref` 為主（見「總結」第 5 點）。

> 📝 補充：以上為依個人筆記整理，如需更精確，可再對照影片的時間點補上（例如：影片中尤雨溪自己的使用習慣）。

## 三、總結

1. **`ref`**：萬用，任何型別都能放，JS 中用 `.value`，可整個重新賦值。
2. **`reactive`**：只限物件，不用 `.value`，但不能整個換掉、解構會失去響應式。
3. 模板中 `ref` 會自動解開，**不用寫 `.value`**。
4. 常見錯誤：忘記 `.value`、替換整個 `reactive` 物件、直接解構 `reactive`。
5. **不確定時一律用 `ref`**，這也是多數團隊與官方文件的建議方向。
