# Frontend-performance-guide

# 🚀 Hướng Dẫn Tối Ưu Hiệu Năng & Debugging Trong Frontend

## 🎯 Mục Tiêu

Tài liệu này chia sẻ các kỹ thuật và best practices giúp bạn:

- Tối ưu hiệu năng rendering.
- Giảm kích thước bundle.
- Tránh render dư thừa.
- Debug hiệu quả hơn trong frontend (Vue/React).

---

## 📦 1. Lazy Loading

### ❌ Không áp dụng:

```ts
import About from "@/pages/About.vue";

const routes = [{ path: "/about", component: About }];
```

- 📉 Mọi route được bundle vào initial JS, dù chưa truy cập.

### ✅ Áp dụng lazy load:

```ts
const routes = [
  { path: "/about", component: () => import("@/pages/About.vue") },
];
```

- 📈 Tối ưu tải đầu tiên, chia nhỏ bundle.

---

## 🧩 2. Code Splitting

### ❌ Không áp dụng:

```ts
import UserManagement from "@/modules/user/UserManagement.vue";
```

### ✅ Áp dụng chunk tên riêng:

```ts
const UserManagement = () =>
  import(
    /* webpackChunkName: "user-module" */ "@/modules/user/UserManagement.vue"
  );
```

- 📈 Giảm bundle chính, dễ cache hơn.

---

## 🧠 3. Memoization

### ❌ Không áp dụng:

```ts
const result = getExpensiveData();
```

### ✅ Dùng computed/memo:

```ts
const result = computed(() => getExpensiveData());
```

- 📈 Không re-calculate nếu input không đổi.

---

## ♻️ 4. Tránh Render Thừa

### ❌ Không nên:

```vue
<BaseSelect :options="[{ label: 'A', value: 1 }]" />
```

### ✅ Tối ưu:

```ts
const options = ref([{ label: "A", value: 1 }]);
```

```vue
<BaseSelect :options="options" />
```

- 📈 Giảm render lại component con.

---

## 🐞 5. Debugging Hiệu Quả

### ✅ Gợi ý:

```ts
console.log("[UserModule] Fetch error:", error);
```

```ts
app.config.errorHandler = (err, instance, info) => {
  console.error(`[Error in ${info}]:`, err);
};
```

- 📈 Trace rõ lỗi, dễ maintain team.

---

## ⚖️ Lợi Ích Khi Áp Dụng vs Hậu Quả Khi Bỏ Qua

| Kỹ thuật          | Nếu áp dụng 🟢                  | Nếu không áp dụng 🔴             |
| ----------------- | ------------------------------- | -------------------------------- |
| Lazy Loading      | Tải nhanh, giảm initial load    | Trang nặng, delay khi load       |
| Code Splitting    | Dễ cache, deploy nhanh          | Bundle lớn, chậm hơn             |
| Memoization       | Giảm tính toán, UI mượt hơn     | Lag, render thừa                 |
| Tránh render thừa | Hiệu năng cao, code sạch        | Component con render lại mỗi lần |
| Debug tốt         | Bắt lỗi dễ, trace logic rõ ràng | Console spam, khó tìm bug        |

---

## ✍️ Viết bởi

**NotAfraiddd - Bảo TNF - TGL Solutions**

Nếu bạn thấy tài liệu này hữu ích, hãy ⭐️ star repo hoặc góp ý thêm nhé!
