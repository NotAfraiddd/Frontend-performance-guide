# 🚀 Frontend Performance Guide

Tối ưu hiệu năng cho ứng dụng **Vue (TypeScript)** và **React / Next.js**.

---

Dưới đây là kết quả của việc áp dụng các kỹ thuật tối ưu hiệu năng cho ứng dụng của mình sau 1 thời gian lăn lộn khá vất vả trong ở các dự án của công ty.

## 🌟 Mục Tiêu

Tài liệu này chia sẻ các kỹ thuật và best practices giúp bạn:

- ✅ Tối ưu hiệu năng rendering.
- ✅ Giảm kích thước bundle.
- ✅ Tránh render dư thừa.

---

## 📦 1. Lazy Loading

**Lazy Loading** (tải lười) là kỹ thuật **trì hoãn việc tải một phần nào đó của ứng dụng cho đến khi nó thật sự cần thiết** (ví dụ: khi người dùng truy cập route đó hoặc component đó được render).

### 🌠 Vue – ❌ Không lazy load:

```ts
import About from "@/pages/About.vue";

const routes = [{ path: "/about", component: About }];
```

- 📉 Mọi route được bundle vào initial JS, dù chưa truy cập.

### ✅ Vue – Áp dụng lazy load:

```ts
const routes = [
  { path: "/about", component: () => import("@/pages/About.vue") },
];
```

- 📈 Tối ưu tải đầu tiên, chia nhỏ bundle.

---

### 🔹 React / Next.js – ❌ Không lazy load:

```ts
import About from "./pages/About";

function App() {
  return <About />;
}
```

- 📉 `About` luôn được tải ngay cả khi chưa được sử dụng.

### ✅ React / Next.js – Áp dụng dynamic import:

```ts
import dynamic from "next/dynamic";

const About = dynamic(() => import("./pages/About"));

function App() {
  return <About />;
}
```

- 📈 `About` chỉ được tải khi cần → giảm initial load.

> ⚠️ Với Next.js: Các route trong thư mục `pages/` **đã tự động lazy load**, nhưng nếu bạn **import component thủ công**, hãy dùng `dynamic()`.

---

## 🧰 2. Code Splitting

**Code Splitting** là kỹ thuật **chia nhỏ mã nguồn JavaScript thành các phần riêng biệt (chunk)** để trình duyệt chỉ tải những phần thật sự cần thiết. Điều này giúp:

- ⏱ Tăng tốc thời gian tải ban đầu (initial load)
- 📦 Giảm kích thước bundle chính
- 🔁 Tối ưu khả năng cache

---

### 🌠 Vue – Khi nào cần Code Splitting?

#### ❌ Không chia nhỏ – tất cả nằm trong bundle chính:

```ts
import UserManagement from "@/modules/user/UserManagement.vue";

const routes = [{ path: "/users", component: UserManagement }];
```

- Khi dùng `import` trực tiếp, file này sẽ luôn nằm trong bundle chính, kể cả khi người dùng chưa truy cập `/users`

#### ✅ Vue – Tách riêng thành chunk (lazy load route):

```ts
const routes = [
  {
    path: "/users",
    component: () =>
      import(
        /* webpackChunkName: "user-module" */ "@/modules/user/UserManagement.vue"
      ),
  },
];
```

- Khi người dùng truy cập `/users`, trình duyệt mới tải `user-module.[hash].js`
- Bạn có thể gán `webpackChunkName` để tên chunk dễ đọc và debug

---

### 🔹 React / Next.js – Khi nào cần Code Splitting?

#### ✅ Tự động chia nhỏ (Next.js `pages/`)

```ts
// pages/users.tsx
export default function UsersPage() {
  return <div>User page</div>;
}
```

- ✅ Mặc định, mỗi file trong `pages/` sẽ là một chunk riêng
- 🪄 Tự động code split theo từng route

#### ✅ Dùng `dynamic()` để chia nhỏ component lớn:

```tsx
import dynamic from "next/dynamic";

const UserManagement = dynamic(
  () => import("@/components/UserManagement"),
  { ssr: false } // nếu không cần SSR
);

export default function Page() {
  return <UserManagement />;
}
```

- `UserManagement` chỉ được tải khi component thực sự render
- Có thể dùng `webpackChunkName` để đặt tên chunk (nếu cần debug):

```tsx
() =>
  import(/* webpackChunkName: "user-module" */ "@/components/UserManagement");
```

---

### 📌 Khi nào nên dùng Code Splitting thủ công?

| Tình huống                                  | Có nên dùng? |
| ------------------------------------------- | ------------ |
| Page ít dùng (admin, setting, report...)    | ✅           |
| Component nặng (Chart, PDF, Map, Editor...) | ✅           |
| Component nhỏ nhưng lặp lại nhiều           | ❌           |

> ⚠️ Đừng lạm dụng `dynamic()` hoặc `() => import()` với các component nhỏ thường xuyên xuất hiện, vì có thể gây chậm do phải tải nhiều file nhỏ.

---

## 🧠 3. Memoization

**Memoization** là kỹ thuật giúp **lưu lại kết quả tính toán tốn kém**, để không cần tính lại mỗi khi component render lại — đặc biệt hữu ích khi xử lý dữ liệu phức tạp, hoặc truyền props xuống component con.

> 🎯 Mục tiêu: **Giữ nguyên reference** (đối với object, array, function) hoặc **tránh gọi lại hàm nặng** nếu dependency không đổi.

---

### 🌠 Vue 3 (Composition API)

#### ❌ KHÔNG memo – Trực tiếp gọi hàm:

```ts
const result = getExpensiveData();
```

- Hàm `getExpensiveData()` sẽ được gọi mỗi lần render → gây tính toán dư thừa.

#### ✅ Dùng `computed()` – Memo giá trị tính toán:

```ts
import { computed } from "vue";

const result = computed(() => getExpensiveData());
```

- `computed` chỉ tính lại khi dependency trong hàm thay đổi.
- Rất hữu ích khi dữ liệu phụ thuộc vào `ref`, `reactive`, v.v.

#### ✅ Dùng `const` cho dữ liệu tĩnh (không reactive):

```ts
const result = expensiveStaticData;
```

- Nếu dữ liệu không cần reactive, bạn có thể dùng `const` để giữ nguyên reference.

---

### 🔹 React / Next.js

#### ✅ Dùng `useMemo()` – Memo object/array hoặc kết quả tính toán:

```tsx
import { useMemo } from "react";

const result = useMemo(() => getExpensiveData(), [dependency]);
```

- `getExpensiveData()` chỉ chạy lại khi `dependency` thay đổi.
- Rất cần thiết nếu `result` là object/array dùng trong props của component con.

#### ✅ Dùng `useCallback()` – Memo callback function:

```tsx
import { useCallback } from "react";

const handleClick = useCallback(() => {
  doSomething();
}, []);
```

- Tránh tạo hàm mới mỗi lần render → giữ reference ổn định → component con không bị render lại.

---

### 📌 Khi nào nên dùng Memoization?

| Trường hợp sử dụng                           | Có nên memo?    |
| -------------------------------------------- | --------------- |
| Hàm tính toán nặng hoặc xử lý dữ liệu lớn    | ✅              |
| Truyền object/array xuống component con      | ✅              |
| Truyền callback function xuống component con | ✅              |
| Dữ liệu tĩnh, không reactive                 | ❌ Dùng `const` |
| Dữ liệu đơn giản, thay đổi thường xuyên      | ❌ Không cần    |

---

### 🧠 Tổng kết

| Framework        | Memo giá trị | Memo function         |
| ---------------- | ------------ | --------------------- |
| **Vue 3**        | `computed()` | Tránh inline function |
| **React / Next** | `useMemo()`  | `useCallback()`       |

> ⚠️ Lạm dụng memoization có thể làm code phức tạp hơn. Chỉ dùng khi có lý do hiệu năng rõ ràng (component chậm, re-render nhiều, logic nặng...).

## ♻️ 4. Tránh Render Dư Thừa

Hiệu suất frontend không chỉ nằm ở tốc độ tải ban đầu, mà còn phụ thuộc vào việc **hạn chế render lại không cần thiết**. Việc truyền object, array hoặc function **trực tiếp** vào props sẽ khiến component con hiểu nhầm rằng dữ liệu đã thay đổi, dù giá trị thực tế không đổi.

---

### 🌠 Vue 3 (Composition API)

#### ❌ Truyền object inline – KHÔNG NÊN:

```vue
<BaseSelect :options="[{ label: 'A', value: 1 }]" />
```

- Mỗi lần parent component re-render, Vue sẽ tạo object mới → reference thay đổi → `BaseSelect` render lại.

#### ✅ Dùng `ref` – NÊN DÙNG:

```ts
import { ref } from "vue";

export default defineComponent({
  setup() {
    const options = ref([{ label: "A", value: 1 }]);
    return { options };
  },
});
```

```vue
<BaseSelect :options="options" />
```

- `ref()` giữ nguyên reference → Vue không re-render `BaseSelect` nếu `options` không thay đổi.

#### ✅ Dữ liệu tĩnh – Dùng `const` ngoài `setup()`:

```ts
const options = [{ label: "A", value: 1 }];

export default defineComponent({
  setup() {
    return { options };
  },
});
```

- `options` chỉ khởi tạo một lần → reference không đổi → hiệu quả như `ref()` nếu dữ liệu tĩnh.

#### ⚠️ KHÔNG NÊN tạo object trong `setup()` mỗi lần:

```ts
setup() {
  const options = [{ label: 'A', value: 1 }];
  return { options };
}
```

- Mỗi lần render là 1 object mới → Vue hiểu là props thay đổi → component con render lại không cần thiết.

---

### 🔹 React / Next.js

#### ❌ Truyền object/array inline – KHÔNG NÊN:

```tsx
<MyComponent options={[{ label: "A", value: 1 }]} />
```

- Mỗi lần render, tạo array mới → `React.memo` không có tác dụng vì shallow compare thấy khác.

#### ✅ Dùng `useMemo` để giữ reference:

```tsx
import { useMemo } from "react";

const options = useMemo(() => [{ label: "A", value: 1 }], []);

<MyComponent options={options} />;
```

- `useMemo` chỉ tạo lại khi dependency thay đổi → giữ reference ổn định.

#### ✅ Với callback function – Dùng `useCallback`:

```tsx
const handleClick = useCallback(() => {
  doSomething();
}, []);

<MyComponent onClick={handleClick} />;
```

- Tránh tạo function mới mỗi lần render → component con không render lại vô ích.

---

## 🧱 5. Tối ưu Component Con

### 🌠 Vue

- Dùng `v-memo`, `defineComponent`, `v-once` nếu dữ liệu không đổi.

### 🔹 React

- Dùng `React.memo` cho component con:

```ts
const MyComponent = React.memo(({ data }) => {
  return <div>{data.title}</div>;
});
```

---

## ⚖️ So Sánh Vue vs React / Next.js

| Kỹ thuật          | Vue (TypeScript)                      | React / Next.js                               |
| ----------------- | ------------------------------------- | --------------------------------------------- |
| Lazy Loading      | `() => import()`                      | `dynamic(() => import())`                     |
| Code Splitting    | Webpack chunk comment                 | Webpack chunk comment + `dynamic()`           |
| Memoization       | `computed`, `ref`                     | `useMemo`, `useCallback`                      |
| Tránh render thừa | Tránh object inline, dùng `ref`       | `useMemo`, tránh object literal               |
| Component tối ưu  | `v-memo`, `v-once`, `defineComponent` | `React.memo`, `PureComponent`, `useMemo`, ... |

---

## ✅ Tổng Kết

| Kỹ thuật          | Nếu áp dụng 🟢               | Nếu không áp dụng 🔴             |
| ----------------- | ---------------------------- | -------------------------------- |
| Lazy Loading      | Tải nhanh, giảm initial load | Trang nặng, delay khi load       |
| Code Splitting    | Dễ cache, deploy nhanh       | Bundle lớn, chậm hơn             |
| Memoization       | Giảm tính toán, UI mượt hơn  | Lag, render thừa                 |
| Tránh render thừa | Hiệu năng cao, code sạch     | Component con render lại mỗi lần |

---

## ✍️ Viết bởi

**NotAfraiddd - Bảo TNF - TGL Solutions**

Nếu bạn thấy tài liệu này hữu ích, hãy ⭐️ star hoặc chia sẻ nhé! Lần đầu mình làm tài liệu này nên có thể có lỗi gì đó, mong nhận được góp ý của mọi người để mình có thể cải thiện hơn nhé.
