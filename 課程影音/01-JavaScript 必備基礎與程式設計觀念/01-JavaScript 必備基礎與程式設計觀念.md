[🏠 回到首頁](../../README.md)

# 01-JavaScript 必備基礎與程式設計觀念

> 學 Vue 之前，必須先搞懂兩件事：
> 1. JavaScript 的「值」與「參考」（Vue 的 `ref`、`reactive` 都建立在這個觀念上）
> 2. 程式為什麼要把「資料、畫面、事件」分開思考（Vue 就是幫我們把這件事做好的框架）

## 1.1 值與參考：物件、陣列的操作特性

### 1. JS 資料型別分為二大類：基本型別與物件型別

| 類別 | 包含的型別 | 範例 |
| --- | --- | --- |
| 基本型別 (Primitive) | `string`、`number`、`boolean`、`undefined`、`null`、`bigint`、`symbol` | `"Kent"`、`10`、`true` |
| 物件型別 (Object) | 除了上面七種以外的都是，例如：物件 `{}`、陣列 `[]`、函式 `function`、日期 `Date` | `{ name: "Kent" }`、`[1, 2, 3]` |

**最重要的差異：**

- **基本型別**：變數裡面放的是「**值本身**」。
- **物件型別**：變數裡面放的是「**記憶體位置（參考 / reference）**」，真正的資料放在另一個地方（堆積 heap）。

> 💡 比喻：
> - 基本型別像「把一張紙條**影印**一份給你」，你塗改你那份，我的不受影響。
> - 物件型別像「把**房子的地址**給你」，你到那個地址把牆壁漆成紅色，我回家也會看到紅牆，因為我們指向同一間房子。

### 2. 變數複製與修改

#### 基本型別：複製的是「值」

```js
let a = 10;
let b = a;
b = 20;
console.log(a); // 10
console.log(b); // 20
```

記憶體指向的情形（每個變數各自擁有一份值）：

```
步驟1  let a = 10;      a ──▶ [ 10 ]

步驟2  let b = a;       a ──▶ [ 10 ]
                        b ──▶ [ 10 ]     ← 複製了一份「值」

步驟3  b = 20;          a ──▶ [ 10 ]     ← 不受影響
                        b ──▶ [ 20 ]
```

**結論：修改 `b` 不會影響 `a`，因為兩者是各自獨立的值。**

#### 物件型別：複製的是「參考（位址）」

```js
const aUser = { id: 10, name: "Kent", skills: ["JavaScript", "HTML"] };
const bUser = aUser;
bUser.name = "Cindy";
console.log(aUser.name); // "Cindy"  ← 也被改了！
console.log(bUser.name); // "Cindy"
console.log(aUser === bUser); // true
```

記憶體指向的情形（兩個變數指向同一個物件）：

```
步驟1  const aUser = {...};
       aUser ──▶ 位址#001 { id:10, name:"Kent", skills:[...] }

步驟2  const bUser = aUser;       （只複製「位址」，沒有複製物件）
       aUser ──┐
               ├──▶ 位址#001 { id:10, name:"Kent", skills:[...] }
       bUser ──┘

步驟3  bUser.name = "Cindy";      （順著位址去改物件內容）
       aUser ──┐
               ├──▶ 位址#001 { id:10, name:"Cindy", skills:[...] }
       bUser ──┘
```

**結論：`aUser` 與 `bUser` 是同一個物件的兩個「名字」，改任何一個，另一個看到的內容也會變。**

> ⚠️ 補充：`const` 鎖住的是「變數裡的位址不能換」，**不是**「物件內容不能改」。
> 所以 `bUser.name = "Cindy"` 可以執行，但 `bUser = {}` 會報錯。

#### 如果真的想要「複製一份新的物件」？

```js
// 淺層複製（第一層是複製，但裡面的物件/陣列仍然共用）
const cUser = { ...aUser };
// 深層複製（完全獨立的一份）
const dUser = structuredClone(aUser);

dUser.skills.push("CSS");
console.log(aUser.skills); // ["JavaScript", "HTML"]  不受影響
```

### 3. 物件修改與重新賦值的差異

```js
const eUser = { name: "Kent" };
const fUser = eUser;

function getUserName1(userObj) {
  userObj.name = "Linda"; // 寫法A：修改物件的內容
}

getUserName1(eUser);
console.log(eUser);        // { name: "Linda" }
console.log(fUser);        // { name: "Linda" }
console.log(eUser === fUser); // true
```

#### 寫法A：「修改」物件內容（順著位址去改）

呼叫函式時，傳進去的是**位址的複製**，所以 `userObj` 與 `eUser`、`fUser` 三者指向同一個物件：

```
呼叫 getUserName1(eUser) 時：

eUser   ──┐
fUser   ──┼──▶ 位址#001 { name: "Kent" }
userObj ──┘

執行 userObj.name = "Linda"：

eUser   ──┐
fUser   ──┼──▶ 位址#001 { name: "Linda" }   ← 同一個物件被改了
userObj ──┘
```

結果：`eUser`、`fUser` 都變成 `{ name: "Linda" }`，`eUser === fUser` 為 `true`。

#### 寫法B：「重新賦值」（讓變數改指向新的物件）

```js
function getUserName2(userObj) {
  userObj = { name: "Linda" }; // 寫法B：重新賦值
}

getUserName2(eUser);
console.log(eUser); // { name: "Kent" }  ← 沒有被改！
console.log(fUser); // { name: "Kent" }
console.log(eUser === fUser); // true
```

記憶體指向的情形：

```
呼叫時：
eUser   ──┐
fUser   ──┼──▶ 位址#001 { name: "Kent" }
userObj ──┘

執行 userObj = { name: "Linda" }：
（建立新物件，userObj 這個「函式內的變數」改指向新位址）

eUser   ──┐
fUser   ──┴──▶ 位址#001 { name: "Kent" }    ← 原物件完全沒動
userObj ─────▶ 位址#002 { name: "Linda" }   ← 新物件，函式結束後就沒人用了
```

#### 差異整理

| | 修改內容 `userObj.name = ...` | 重新賦值 `userObj = {...}` |
| --- | --- | --- |
| 做了什麼 | 順著位址，改**同一個物件**的內容 | 讓 `userObj` 改指向**另一個新物件** |
| 原本的物件 | 被改變 | **不變** |
| 外面的 `eUser` | 會看到變化 | 看不到變化 |

> 💡 口訣：**「改內容」會影響大家；「換位址」只影響自己。**

### 🔗 為什麼這跟 Vue 有關？

- Vue 的 `reactive` / `ref` 是對「物件」做監聽，**修改內容**（`state.name = "x"`）會觸發畫面更新。
- 若把整個 `reactive` 物件**重新賦值**，會讓變數指向新物件，可能導致**失去響應性**。
- 傳遞 props、陣列操作（`push`、`splice`）也都與「共用同一份參考」有關。

---

## 1.2 關注點分離與前端程式結構

### 1. 關注點分離 (Separation of Concerns)

> 將程式中**不同責任分開思考與處理**，讓我們更容易知道每一段程式碼正在負責什麼。

一個前端畫面通常有三種「責任」：

| 關注點 | 負責什麼 | 計數器範例 |
| --- | --- | --- |
| **資料 (State)** | 現在的狀態是什麼 | `count` 目前的數字 |
| **畫面 (View)** | 資料長什麼樣子 | `<div>` 顯示數字、`<button>` |
| **事件處理 (Action)** | 使用者做了什麼，資料要怎麼變 | 按下按鈕後 `count + 1` |

分開思考的好處：

- 要改畫面 → 只看畫面的部分
- 要改計算邏輯 → 只看資料與事件的部分
- 程式變大時，不會全部纏在一起（俗稱「義大利麵程式碼」）

### 2. 計數器：JS 與 Vue 的比較

範例檔案：[JS 版 counter-js.html](counter-js.html)、[Vue 版 counter-vue.html](counter-vue.html)

#### 原生 JavaScript 版

```html
<!-- 畫面 -->
<div>
  <div id="count"></div>
  <button type="button" id="add">+1</button>
</div>

<script>
  // 資料
  let count = 0;

  // 畫面元素
  const countEl = document.getElementById("count");
  const addBtn = document.getElementById("add");

  // 畫面渲染
  function render() {
    countEl.textContent = count;
  }

  // 事件處理
  addBtn.addEventListener("click", () => {
    count++;
    render(); // ← 必須「手動」叫畫面更新
  });

  render();
</script>
```

#### Vue 版

這個版本使用 CDN 引入 Vue 3，不需要安裝任何工具，直接用瀏覽器開啟檔案就能執行。

```html
<!-- 畫面 -->
<div id="app">
  <div>{{ count }}</div>
  <button type="button" @click="add">+1</button>
</div>

<!-- 引入 Vue 3 (CDN) -->
<script src="https://unpkg.com/vue@3.5.13/dist/vue.global.prod.js"></script>
<script>
  const { createApp, ref } = Vue;

  createApp({
    setup() {
      // 資料
      const count = ref(0);
      // 事件處理
      const add = () => {
        count.value++; // ← 只改資料，畫面自動更新
      };
      // 提供給模板使用
      return { count, add };
    },
  }).mount("#app"); // 把 Vue 掛載到 #app 這個區塊
</script>
```

程式碼逐段說明：

| 程式碼 | 說明 |
| --- | --- |
| `<script src="...vue.global.prod.js">` | 從 CDN 載入 Vue，載入後會產生全域物件 `Vue` |
| `const { createApp, ref } = Vue;` | 從 `Vue` 取出要用的功能：`createApp` 建立應用程式、`ref` 建立響應式資料 |
| `ref(0)` | 建立一個「響應式」的數字，初始值 0；資料改變時，畫面會自動更新 |
| `setup()` | 放資料與函式的地方，最後 `return` 的東西，模板才能使用 |
| `.mount("#app")` | 告訴 Vue：「請管理 `#app` 這個區塊」 |
| `{{ count }}` | 模板語法，把 `count` 的值顯示出來 |
| `@click="add"` | 綁定點擊事件，等於原生的 `addEventListener("click", add)` |

> 📝 注意：`ref` 的值在 `<script>` 中要用 `.value` 讀寫（`count.value++`），在模板中則直接寫 `count`，Vue 會自動處理。

#### 二者差異

| 比較項目 | 原生 JavaScript | Vue |
| --- | --- | --- |
| 寫法風格 | **命令式**：一步一步告訴瀏覽器「怎麼做」 | **宣告式**：只描述「畫面應該長什麼樣」 |
| 取得畫面元素 | 要自己 `getElementById` 找元素 | 不需要，用 `{{ count }}` 直接綁定 |
| 綁定事件 | `addEventListener("click", ...)` | `@click="add"` |
| 更新畫面 | 資料改了之後，要**手動**呼叫 `render()` | 資料改了，**自動**更新畫面（響應式） |
| 資料與畫面的關係 | 兩者分開，要靠程式碼手動同步 | 兩者自動連動 |
| 忘記更新時 | 資料變了但畫面沒變（常見 bug） | 不會發生這種問題 |
| 程式變複雜時 | 需要管理的 `render()` 與 DOM 操作愈來愈多 | 仍只需專注在資料上 |

#### 重點觀察

1. **關注點分離的程度不同**
   - 原生 JS：資料、DOM 操作、畫面更新**混在同一個 `<script>`**，開發者要自己負責「同步」。
   - Vue：開發者只管**資料 + 事件**，「資料如何變成畫面」交給框架處理。

2. **心智負擔不同**
   - 原生 JS 要想：「按下去後，我要找到哪個元素？改它的什麼屬性？」
   - Vue 要想：「按下去後，資料要變成什麼？」

3. **核心概念：`畫面 = f(資料)`**
   畫面是資料的結果。資料變了，畫面自然跟著變。這也是接下來學習 Vue 的出發點。

---

## ✅ 本章重點回顧

1. 基本型別存「值」，物件型別存「位址（參考）」。
2. 複製物件變數 = 複製位址，兩個變數會指向**同一個**物件。
3. **修改內容**會影響所有指向該物件的變數；**重新賦值**只會改變那個變數自己的指向。
4. `const` 只保證「位址不變」，不保證「內容不變」。
5. 關注點分離：把**資料、畫面、事件**分開思考。
6. Vue 讓我們只專注在「資料」，畫面由框架自動同步。

---

## 📝 練習題

> 範圍：1.1 值與參考、1.2 關注點分離與前端程式結構
> 建議做法：先自己作答，再展開「答案與詳解」對照。

---

### 一、選擇題（共 8 題）

#### Q1. 下列哪一個**不是**基本型別（Primitive）？

- (A) `"Kent"`
- (B) `null`
- (C) `[1, 2, 3]`
- (D) `10n`（bigint）

<details>
<summary>答案與詳解</summary>

**答案：(C)**

陣列屬於物件型別。基本型別共七種：`string`、`number`、`boolean`、`undefined`、`null`、`bigint`、`symbol`。(D) 的 `10n` 是 `bigint`，(B) `null` 也是基本型別。

</details>

#### Q2. 執行下列程式後，`a` 與 `b` 的值為何？

```js
let a = 10;
let b = a;
b = 20;
console.log(a, b);
```

- (A) `10 10`
- (B) `20 20`
- (C) `10 20`
- (D) `20 10`

<details>
<summary>答案與詳解</summary>

**答案：(C)**

基本型別複製的是「值」，`b = a` 之後兩者各自擁有一份 `10`，修改 `b` 不會影響 `a`。

</details>

#### Q3. 執行下列程式後，輸出為何？

```js
const aUser = { name: "Kent" };
const bUser = aUser;
bUser.name = "Cindy";
console.log(aUser.name, aUser === bUser);
```

- (A) `"Kent" false`
- (B) `"Cindy" true`
- (C) `"Kent" true`
- (D) `"Cindy" false`

<details>
<summary>答案與詳解</summary>

**答案：(B)**

物件型別複製的是「參考（位址）」，`aUser` 與 `bUser` 指向同一個物件，所以改 `bUser.name`，`aUser.name` 也跟著變，且 `===` 比較的是位址，結果為 `true`。

</details>

#### Q4. 關於 `const` 的敘述，何者正確？

- (A) `const` 宣告的物件，內容完全不能修改
- (B) `const` 鎖住的是變數裡的「位址」，不能重新賦值，但物件內容可以改
- (C) `const` 只適用於基本型別
- (D) `const` 宣告的變數可以重新賦值成新物件

<details>
<summary>答案與詳解</summary>

**答案：(B)**

`const user = {}` 之後，`user.name = "x"` 可以執行（改內容），但 `user = {}` 會報錯（換位址）。(A)(D) 與此相反，(C) 錯誤，`const` 可用於任何型別。

</details>

#### Q5. 執行下列程式後，`console.log(eUser)` 輸出為何？

```js
const eUser = { name: "Kent" };

function change(userObj) {
  userObj = { name: "Linda" };
}

change(eUser);
console.log(eUser);
```

- (A) `{ name: "Linda" }`
- (B) `{ name: "Kent" }`
- (C) `undefined`
- (D) 報錯

<details>
<summary>答案與詳解</summary>

**答案：(B)**

函式內的 `userObj = {...}` 是「重新賦值」，只是讓函式內的 `userObj` 改指向新物件，原本的 `eUser` 指向的物件完全沒被動到。口訣：**改內容會影響大家；換位址只影響自己。**

</details>

#### Q6. 下列程式輸出為何？

```js
const aUser = { name: "Kent", skills: ["JS", "HTML"] };
const cUser = { ...aUser };
cUser.name = "Amy";
cUser.skills.push("CSS");
console.log(aUser.name, aUser.skills);
```

- (A) `"Kent" ["JS", "HTML"]`
- (B) `"Amy" ["JS", "HTML", "CSS"]`
- (C) `"Kent" ["JS", "HTML", "CSS"]`
- (D) `"Amy" ["JS", "HTML"]`

<details>
<summary>答案與詳解</summary>

**答案：(C)**

展開運算子 `{ ...aUser }` 是**淺層複製**：第一層屬性（`name`）是各自獨立的值，所以 `aUser.name` 維持 `"Kent"`；但 `skills` 是陣列（物件），複製的是位址，兩者共用同一個陣列，`push("CSS")` 會同時影響 `aUser.skills`。想完全獨立要用 `structuredClone`。

</details>

#### Q7. 下列哪一項**最符合**「關注點分離」中三種責任的對應？

- (A) 資料＝`<button>`、畫面＝`count`、事件＝`<div>`
- (B) 資料＝`count`、畫面＝`<div>` 與 `<button>`、事件＝按下按鈕後 `count + 1`
- (C) 資料＝按下按鈕、畫面＝`count`、事件＝`<div>`
- (D) 資料、畫面、事件都應寫在同一處以方便閱讀

<details>
<summary>答案與詳解</summary>

**答案：(B)**

- 資料（State）：現在的狀態，例如 `count`
- 畫面（View）：資料長什麼樣子，例如 `<div>`、`<button>`
- 事件處理（Action）：使用者做了什麼、資料怎麼變

(D) 剛好相反，全部纏在一起就是「義大利麵程式碼」。

</details>

#### Q8. 關於原生 JavaScript 與 Vue 計數器的差異，何者**錯誤**？

- (A) 原生 JS 是命令式，Vue 是宣告式
- (B) 原生 JS 資料改變後，需手動呼叫 `render()` 更新畫面
- (C) Vue 的 `ref` 在 `<script>` 中讀寫要用 `.value`
- (D) Vue 中資料改變後，仍需手動操作 DOM 才能更新畫面

<details>
<summary>答案與詳解</summary>

**答案：(D)**

Vue 具備響應式，資料改變後畫面會自動更新，不需手動操作 DOM。核心概念是 **畫面 = f(資料)**。

</details>

---

### 二、實作題（共 2 題）

#### 實作 1：物件複製與修改

請依照下列需求，補完程式碼，使最後的 `console.log` 輸出與註解相同。

```js
const original = {
  name: "Kent",
  skills: ["JavaScript", "HTML"],
};

// (1) 建立 sameRef：與 original 共用同一個物件
// (2) 建立 shallow：用展開運算子做淺層複製
// (3) 建立 deep：做深層複製，與 original 完全獨立

sameRef.name = "Cindy";
shallow.skills.push("CSS");
deep.skills.push("Vue");

console.log(original.name);   // "Cindy"
console.log(original.skills); // ["JavaScript", "HTML", "CSS"]
console.log(deep.skills);     // ["JavaScript", "HTML", "Vue"]
console.log(original === sameRef); // true
```

並回答：為什麼 `deep.skills.push("Vue")` 不會影響 `original.skills`？

<details>
<summary>答案與詳解</summary>

```js
const original = {
  name: "Kent",
  skills: ["JavaScript", "HTML"],
};

const sameRef = original;                 // (1) 複製的是位址
const shallow = { ...original };          // (2) 淺層複製
const deep = structuredClone(original);   // (3) 深層複製

sameRef.name = "Cindy";        // 改到 original 本身
shallow.skills.push("CSS");    // skills 陣列與 original 共用
deep.skills.push("Vue");       // deep 擁有獨立的 skills 陣列

console.log(original.name);        // "Cindy"
console.log(original.skills);      // ["JavaScript", "HTML", "CSS"]
console.log(deep.skills);          // ["JavaScript", "HTML", "Vue"]
console.log(original === sameRef); // true
```

**詳解：**

1. `sameRef = original`：只複製位址，兩者是同一個物件的兩個名字，所以改 `sameRef.name` 會影響 `original`。
2. `shallow`：第一層屬性被複製，但 `skills` 是陣列，複製的是位址，所以 `shallow.skills` 與 `original.skills` 是同一個陣列。注意 `shallow.name` 之後若修改，則不會影響 `original`。
3. `deep`：`structuredClone` 連巢狀的物件、陣列都建立新的一份，與 `original` 完全獨立，因此 `push("Vue")` 只影響 `deep.skills`。

</details>

#### 實作 2：把原生計數器改寫成「資料 / 畫面 / 事件」分離的結構

請寫一個**原生 JavaScript** 版本的「+1 / −1 / 歸零」計數器，要求：

1. HTML 有一個顯示數字的 `<div id="count">` 與三個按鈕：`+1`、`-1`、`重置`。
2. 程式碼用註解明確分成：**資料**、**畫面元素**、**畫面渲染（`render`）**、**事件處理**四段。
3. 每次資料改變後都要更新畫面。
4. 完成後，再簡短說明若改用 Vue 會少寫哪些程式碼。

<details>
<summary>答案與詳解</summary>

**原生 JavaScript 參考解答：**

```html
<div>
  <div id="count"></div>
  <button type="button" id="add">+1</button>
  <button type="button" id="minus">-1</button>
  <button type="button" id="reset">重置</button>
</div>

<script>
  // ===== 資料 (State) =====
  let count = 0;

  // ===== 畫面元素 (View) =====
  const countEl = document.getElementById("count");
  const addBtn = document.getElementById("add");
  const minusBtn = document.getElementById("minus");
  const resetBtn = document.getElementById("reset");

  // ===== 畫面渲染 =====
  function render() {
    countEl.textContent = count;
  }

  // ===== 事件處理 (Action) =====
  addBtn.addEventListener("click", () => {
    count++;
    render();
  });
  minusBtn.addEventListener("click", () => {
    count--;
    render();
  });
  resetBtn.addEventListener("click", () => {
    count = 0;
    render();
  });

  render(); // 初始畫面
</script>
```

**Vue 版本會省略的部分：**

```html
<div id="app">
  <div>{{ count }}</div>
  <button type="button" @click="count++">+1</button>
  <button type="button" @click="count--">-1</button>
  <button type="button" @click="count = 0">重置</button>
</div>
```

- 不需要 `getElementById` 取元素（用 `{{ count }}` 綁定）。
- 不需要 `addEventListener`（用 `@click`）。
- **不需要手動呼叫 `render()`**：資料（`ref`）變了，畫面自動更新。

**評分重點：** 資料、畫面、事件三段是否分開；每個事件處理後是否有呼叫 `render()`（漏掉就會出現「資料變了、畫面沒變」的 bug）。

</details>
