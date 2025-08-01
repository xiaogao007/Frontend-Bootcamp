> “为什么我的 Vue 项目本地跑得好好的，上线一刷新就 404？”  
> “为什么有的网站带 #，有的又不带？”  
> 今天用 5 分钟 + 30 行代码，把这两个问号拉直！

---

## 一、先上车：两条路线长啥样？

| 模式   | 典型 URL                          | 刷新后 | SEO  | 颜值 | 后端配合 |
|--------|-----------------------------------|--------|------|------|----------|
| **Hash**   | `https://music.163.com/#/playlist` | ✅ 不 404 | 差   | 有 # | 不用管 |
| **History**| `https://twitter.com/home`         | ❌ 可能 404 | 好   | 清爽 | 要重写 |

一句话总结：  
**Hash 模式：老派但稳；History 模式：高颜值但需后端兜底。**

---

## 二、Hash 模式：URL 里的“锚点魔法”

### 1️⃣ 原理

- URL `#` 后面的内容叫 **hash**，变化不会触发浏览器重新请求；
- 监听 `hashchange` 事件即可切换组件，**0 后端成本**。

### 2️⃣ 30 行“手搓” Hash 路由

```html
<!-- index.html -->
<body>
  <ul>
    <li><a href="#/home">Home</a></li>
    <li><a href="#/about">About</a></li>
  </ul>
  <div id="app">加载中...</div>

  <script>
    const routes = {
      '/home': () => '<h2>🏠 Home 页面</h2>',
      '/about': () => '<h2>👋 About 页面</h2>'
    };

    function router() {
      const path = location.hash.slice(1) || '/home';
      document.getElementById('app').innerHTML = routes[path]();
    }

    window.addEventListener('hashchange', router);
    window.addEventListener('load', router); // 首次加载
  </script>
</body>
```

**彩蛋**：打开 `index.html#/about` 直接定位到 About，**刷新不 404**，因为浏览器只认 `#` 前面的路径。

---

## 三、History 模式：URL 界的“颜值担当”

### 1️⃣ 原理

- 使用 HTML5 的 `pushState()` / `replaceState()` 修改 URL，**不刷新页面**；
- 但直接访问或刷新时，浏览器会真的去请求这个路径，**服务器必须返回同一个 SPA 入口**。

### 2️⃣ 30 行“手搓” History 路由

```html
<!-- index.html -->
<body>
  <ul>
    <li><a href="/home">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
  <div id="app">加载中...</div>

  <script>
    const routes = {
      '/home': () => '<h2>🏠 Home 页面</h2>',
      '/about': () => '<h2>👋 About 页面</h2>'
    };

    function router() {
      const path = location.pathname;
      document.getElementById('app').innerHTML = routes[path]();
    }

    // 拦截所有 <a> 跳转
    document.addEventListener('click', (e) => {
      if (e.target.tagName === 'A') {
        e.preventDefault();
        history.pushState(null, null, e.target.getAttribute('href'));
        router();
      }
    });

    window.addEventListener('popstate', router); // 浏览器前进后退
    window.addEventListener('DOMContentLoaded', router);
  </script>
</body>
```

---

## 四、实战：Vue Router 一行切换

```js
// router/index.js
import { createRouter, createWebHistory, createWebHashHistory } from 'vue-router'

const routes = [
  { path: '/', component: () => import('@/views/Home.vue') },
  { path: '/about', component: () => import('@/views/About.vue') }
]

// 一键切换：createWebHashHistory 或 createWebHistory
export default createRouter({
  history: createWebHistory(), // 上线记得配 Nginx！
  routes
})
```

---

## 五、后端兜底锦囊（History 模式必看）

| 服务器 | 重写规则示例 |
|--------|--------------|
| **Nginx** | `try_files $uri $uri/ /index.html;` |
| **Apache** | `FallbackResource /index.html` |
| **Vercel** | 自动帮你配好，无需动手 |

---

## 六、一句话面试回答模板

> “Hash 模式利用 `#` 后面的变化监听路由，**兼容 IE8**，刷新不 404；  
> History 模式用 `pushState` 保持 URL 清爽，**需要服务器将所有路径指回 `index.html`**。  
> 在 Vue 中通过 `createRouter` 的 `history` 参数一键切换。”

---

## 🏁 结语

Hash 像一条老旧的省道——坑坑洼洼但从不堵车；  
History 像一条新修的高速——风景优美但得记得带「通行证」（后端配置）。  
根据项目场景、浏览器兼容和 SEO 需求，选最适合的那条路，才能把用户体验开到 120 km/h！