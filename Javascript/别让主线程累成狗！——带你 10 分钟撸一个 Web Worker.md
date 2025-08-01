

### 🐶 开场白：主线程的悲惨日常

想象一下，你正在快乐地刷着网页，突然点了个「生成年度账单」按钮，页面瞬间卡成 PPT，连滚动条都开始跳机械舞。  
罪魁祸首就是：**JavaScript 主线程被一堆计算活埋了**。

这时候，你需要一个**影子分身**——Web Worker！它能在浏览器后台悄悄干活，不抢主线程的饭碗，让页面永远丝滑得像德芙广告。

---

### 🧬 什么是 Web Worker？

一句话：  
> **Web Worker 是浏览器提供的「后台线程 API」**，让你把重计算、加密、大文件处理等脏活累活丢给它，主线程继续专心伺候用户。

**特点**：
- 运行在**独立线程**，不阻塞 UI  
- **无法操作 DOM**（防止你乱来）  
- 通过**消息机制**与主线程通信（像异地恋，靠微信传情）

---

### 🔥 实战：用 Worker 算 1 亿个数之和

#### 1️⃣ 主线程（index.html）

```html
<!DOCTYPE html>
<html>
<body>
  <button onclick="startWorker()">开始计算！</button>
  <p id="result">结果会出现在这里</p>

  <script>
    function startWorker() {
      // 1. 创建 Worker（注意同源限制）
      const worker = new Worker('./worker.js');

      // 2. 发消息：把活丢给 Worker
      worker.postMessage({ count: 1e8 });

      // 3. 收结果：Worker 算完会回传
      worker.onmessage = (e) => {
        document.getElementById('result').textContent = `总和：${e.data}`;
        worker.terminate(); // 干完活就让它下班
      };

      // 4. 监听错误（防止背锅）
      worker.onerror = (err) => console.error('Worker 崩溃：', err);
    }
  </script>
</body>
</html>
```

#### 2️⃣ Worker 线程（worker.js）

```javascript
// 偷偷在后台算 1 亿个数之和
self.onmessage = function (e) {
  const { count } = e.data;
  let sum = 0;

  // 故意算慢点，让你看清主线程不卡
  for (let i = 0; i < count; i++) {
    sum += i;
  }

  // 把结果传回主线程
  self.postMessage(sum);
};
```

#### 3️⃣ 效果对比
- **不用 Worker**：页面卡死 3 秒，鼠标转圈  
- **用 Worker**：页面秒开，进度条优雅加载（甚至能边算边刷抖音）

---

### 🎪 进阶玩法：SharedArrayBuffer + Atomics（多线程抢红包）

如果你想让多个 Worker 同时读写同一块内存（比如多人协作画板），可以用 `SharedArrayBuffer` + `Atomics` 实现「无锁并发」。  
（PS：需要开启跨域隔离，生产环境需 HTTPS）

```javascript
// 主线程创建共享内存
const sharedBuffer = new SharedArrayBuffer(4); // 4 字节
const sharedArray = new Int32Array(sharedBuffer);

// Worker 里原子加 1
Atomics.add(sharedArray, 0, 1);
```

---

### 🐱‍🏍 彩蛋：Worker 也能玩花样

| 类型         | 用途                          | 示例代码片段                    |
|--------------|-------------------------------|---------------------------------|
| **Inline Worker** | 直接写在 `<script>` 标签里，无需文件 | `const blob = new Blob([code], {type: 'application/javascript'});` |
| **Service Worker** | 离线缓存、PWA 黑科技         | `navigator.serviceWorker.register('sw.js')` |
| **Audio Worklet** | 音频线程（实时处理麦克风）    | 用于 Web Audio API 的音频线程    |

---

### 🌈 结语：把重活交给 Worker，主线程只负责貌美如花

下次遇到：
- 前端 Excel 导入导出（几十万行数据）
- 图片压缩/滤镜（Canvas 像素级计算）
- 加密解密（比如 Web3 钱包）

**第一反应：上 Worker！**  
毕竟，**狗都能累吐舌头，但浏览器主线程不能。**  

> “Worker 不是银弹，但它是主线程的呼吸机。”  
> —— 某位不想加班的前端

---

📦 **源码仓库**：[GitHub Demo 点这里](https://github.com/xiaogao007/web-worker-demo)  
💬 **评论区提问**：你准备用 Worker 优化哪个功能？