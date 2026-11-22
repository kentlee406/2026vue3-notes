[🏠 回到首頁](../README.md)

# 05-SFC（單文件元件）

## 一、SFC 是什麼？

### 1-1 全名與意義

| 項目     | 內容                                                     |
| -------- | -------------------------------------------------------- |
| 英文全名 | **Single-File Component**（簡稱 SFC）                    |
| 中文全名 | **單文件元件**（亦稱「單檔案元件」）                     |
| 副檔名   | `.vue`                                                   |
| 意義     | 把一個元件的**模板、邏輯、樣式**集中寫在**同一個檔案**中 |

> 一句話：**一個 `.vue` 檔 = 一個元件，HTML、JS、CSS 三合一。**

SFC 並不是瀏覽器原生支援的格式，必須透過 **Vite**（搭配 `@vitejs/plugin-vue`）或 Vue CLI / webpack（`vue-loader`）等建構工具**編譯**成一般的 JavaScript 與 CSS 後才能執行。

### 1-2 為什麼會出現 SFC？

傳統做法（如 Vue 2 早期的 `Vue.component()` 全域註冊）有以下問題：

- 模板寫在字串或 HTML 裡，**沒有語法高亮**
- 不支援元件級 CSS，樣式容易互相污染
- 沒有建構步驟，無法使用 ES Module、預處理器（Sass、TypeScript）
- 結構、邏輯、樣式分散在不同檔案，維護時要跳來跳去

SFC 的設計理念是**「關注點分離」不等於「檔案類型分離」**：同一個元件的模板、邏輯、樣式本來就高度相關，放在一起反而更好維護。

## 二、SFC 的基本結構

一個 SFC 由三種區塊（block）組成：

```vue
<script setup>
import { ref } from "vue";

const count = ref(0);
</script>

<template>
  <button @click="count++">點了 {{ count }} 次</button>
</template>

<style scoped>
button {
  color: #42b883;
}
</style>
```

| 區塊                          | 用途                            | 說明                                                               |
| ----------------------------- | ------------------------------- | ------------------------------------------------------------------ |
| `<template>`                  | 模板（HTML 結構）               | 每個 SFC 最多一個頂層 `<template>`；內容會被編譯成 render function |
| `<script>` / `<script setup>` | 邏輯（JavaScript / TypeScript） | 可使用 `lang="ts"`；`<script setup>` 是 Composition API 的語法糖   |
| `<style>`                     | 樣式（CSS）                     | 可有多個；支援 `scoped`、`module`、`lang="scss"` 等                |

另外還有**自訂區塊**（如 `<i18n>`、`<docs>`），需搭配工具處理。

### 2-1 重要的 `<style>` 屬性

```vue
<!-- 1. scoped：樣式只作用於當前元件 -->
<style scoped>
.title {
  color: red;
}
</style>

<!-- 2. module：CSS Modules，透過 $style 存取 -->
<style module>
.title {
  color: blue;
}
</style>

<!-- 3. lang：使用預處理器 -->
<style lang="scss" scoped>
.box {
  .title {
    color: green;
  }
}
</style>
```

> `scoped` 的原理：編譯時替元素加上唯一的 `data-v-xxxx` 屬性，CSS 選擇器也會附加該屬性，達到樣式隔離。

## 三、SFC 的特色

1. **元件化封裝**：模板、邏輯、樣式內聚在一個檔案，一個檔案就是一個可重用單位
2. **樣式隔離**：`<style scoped>` 避免全域樣式污染
3. **編譯期優化**：Vue 編譯器可在編譯時分析模板，做**靜態提升（hoisting）**、**Patch Flags**、**Tree-shaking** 等，提升執行效能
4. **完整的開發體驗**：語法高亮、IDE 支援（Volar / Vue - Official）、型別檢查、HMR 熱更新
5. **支援預處理器**：TypeScript、Pug、Sass / Less / Stylus 都可直接在對應區塊使用
6. **一個元件一個檔案**：命名即元件名（如 `TodoItem.vue`），專案結構一目了然
7. **可搭配 `<script setup>`**：程式碼更精簡，且效能更好

## 四、優缺點分析

### 4-1 優點

| 優點           | 說明                                                    |
| -------------- | ------------------------------------------------------- |
| 高內聚、易維護 | 改一個元件只需開一個檔案，不用在 HTML / JS / CSS 間切換 |
| 重用性高       | 元件可獨立匯入、組合，利於團隊分工                      |
| 樣式不衝突     | `scoped` 或 CSS Modules 提供元件級樣式隔離              |
| 開發體驗佳     | 工具鏈完整：語法高亮、型別推論、HMR、ESLint / Prettier  |
| 效能較佳       | 模板預先編譯，執行時不需要再編譯模板（不需帶 compiler） |
| 支援現代語法   | 可直接使用 ES Module、TypeScript、SCSS 等               |

### 4-2 缺點

| 缺點                 | 說明                                                                       |
| -------------------- | -------------------------------------------------------------------------- |
| **必須有建構步驟**   | 瀏覽器無法直接執行 `.vue`，需 Vite / webpack 等工具，不能只靠 CDN          |
| 學習門檻             | 初學者需了解建構工具、模組化、專案結構                                     |
| 檔案可能過長         | 邏輯複雜的元件，三個區塊擠在同一檔案會變很長，需適度拆分元件或 composable  |
| 傳統分離習慣者不適應 | 習慣 HTML / CSS / JS 分開寫的人需要轉換思維                                |
| `scoped` 的限制      | 子元件內部樣式不易修改（需 `:deep()`）；大量 scoped 會增加屬性與選擇器權重 |
| 綁定框架             | `.vue` 是 Vue 專屬格式，元件無法直接給其他框架使用                         |

> 補充：若不想拆成三區塊，SFC 也支援以 `src` 屬性匯入外部檔案：
> `<script src="./foo.js"></script>`、`<style src="./foo.css"></style>`。

## 五、SFC 與 Composition API 的關係

### 5-1 先釐清：兩者不是同一層級的東西

| 比較           | SFC                                | Composition API                                       |
| -------------- | ---------------------------------- | ----------------------------------------------------- |
| 本質           | **檔案格式 / 元件的組織方式**      | **撰寫元件邏輯的 API 風格**                           |
| 解決的問題     | 元件的**模板、邏輯、樣式如何放置** | 元件內的**邏輯如何組織與重用**                        |
| 對應位置       | 整個 `.vue` 檔案                   | `<script>` 區塊內部的寫法                             |
| 是否可獨立使用 | ✅ 可（搭配 Options API）          | ✅ 可（不使用 SFC，用 `defineComponent` + `setup()`） |

> 一句話：**SFC 是「容器」，Composition API 是「容器內 script 的寫法」。兩者是正交（互相獨立）的，但搭配起來最強。**

### 5-2 三種組合方式

```vue
<!-- ① SFC + Options API（Vue 2 風格，Vue 3 仍支援） -->
<script>
export default {
  data() {
    return { count: 0 };
  },
  methods: {
    add() {
      this.count++;
    },
  },
};
</script>
```

```vue
<!-- ② SFC + Composition API（setup() 函式） -->
<script>
import { ref } from "vue";

export default {
  setup() {
    const count = ref(0);
    const add = () => count.value++;
    return { count, add }; // 必須手動 return 給模板使用
  },
};
</script>
```

```vue
<!-- ③ SFC + Composition API（<script setup>，官方推薦） -->
<script setup>
import { ref } from "vue";

const count = ref(0);
const add = () => count.value++; // 頂層變數自動暴露給模板，不用 return
</script>
```

### 5-3 `<script setup>`：兩者的橋樑

`<script setup>` 是**只能在 SFC 中使用**的編譯期語法糖，專為 Composition API 設計：

| 特色                  | 說明                                                         |
| --------------------- | ------------------------------------------------------------ |
| 免 `return`           | 頂層宣告的變數、函式、import 的元件，模板可直接使用          |
| 免註冊元件            | `import` 進來的元件可直接在模板使用，不用寫 `components: {}` |
| 更簡潔的 Props / Emit | 使用 `defineProps()`、`defineEmits()` 編譯器巨集             |
| 更好的 TS 支援        | 型別可直接推論，Props 型別宣告更自然                         |
| 更好的執行效能        | 模板被編譯成同作用域的 inline render function，少一層代理    |

```vue
<script setup>
import { ref } from "vue";
import ChildCard from "./ChildCard.vue"; // 直接使用，無需註冊

const props = defineProps({ title: String });
const emit = defineEmits(["update"]);

const count = ref(0);
</script>

<template>
  <ChildCard :title="props.title" @click="emit('update', count)" />
</template>
```

### 5-4 為什麼說「搭配起來最強」？

- **SFC 解決「結構」**：把一個元件的三個部分收在一起
- **Composition API 解決「邏輯」**：用 `ref`、`computed`、`watch`、生命週期函式組織邏輯，並可抽成 **composable**（如 `useCounter()`）跨元件重用
- **`<script setup>` 把兩者縫合**：讓 SFC 內的 Composition API 寫法最精簡

```text
SFC（.vue 檔案）
├─ <template>          ← 畫面
├─ <script setup>      ← Composition API 寫在這裡
│    ├─ ref / reactive
│    ├─ computed / watch
│    └─ 呼叫 composable（useXxx）
└─ <style scoped>      ← 樣式
```

## 六、重點整理

| 問題                        | 答案                                                                                                    |
| --------------------------- | ------------------------------------------------------------------------------------------------------- |
| SFC 全名？                  | Single-File Component／單文件元件（`.vue`）                                                             |
| 意義？                      | 一個檔案包含元件的模板、邏輯、樣式                                                                      |
| 需要建構工具嗎？            | 需要（Vite / webpack 編譯）                                                                             |
| 最大優點？                  | 高內聚、樣式隔離、開發體驗佳、編譯期優化                                                                |
| 最大缺點？                  | 必須有建構步驟、檔案可能過長、綁定 Vue                                                                  |
| 與 Composition API 的關係？ | 獨立概念：SFC 是檔案格式，Composition API 是 script 的寫法；`<script setup>` 是兩者結合的官方推薦語法糖 |
