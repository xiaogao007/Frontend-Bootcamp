  
在 JavaScript 的世界里，`if-else` 就像一把瑞士军刀：简单、万能、随处可见。但业务一旦膨胀，层层嵌套的判断语句就会像一团乱麻，剪不断、理还乱。好消息是，这门灵活的语言给我们提供了不少“换刀”方案——既保留语义，又让代码看上去像精心排版的文章，而不是绕口令。

下面分享 5 种“无痛替代”思路，配合代码示例，帮你把“面条状”条件判断梳理成“鱼骨图”。

---

### 1. 一行解决战斗：三元运算

当逻辑只有两个结果，并且只是为了给变量赋值时，三元运算符就是浓缩咖啡版 `if-else`。

**老写法：**

```js
let tip;
if (isVip) {
  tip = 0.2;
} else {
  tip = 0.1;
}
```

**新写法：**

```js
const tip = isVip ? 0.2 : 0.1;
```

一行写完，读代码的人再也不用上下翻屏找 `else` 藏在哪儿。

---

### 2. 字典式查找：用对象/Map 做“翻译表”

当条件多、但每个条件只对应一个固定返回值时，把“判断”变成“查字典”，既直观又易扩展。

**老写法：**

```js
function getIcon(type) {
  if (type === 'pdf') return '📄';
  if (type === 'img') return '🖼️';
  if (type === 'zip') return '🗜️';
  return '❓';
}
```

**新写法：**

```js
const iconMap = {
  pdf: '📄',
  img: '🖼️',
  zip: '🗜️',
};

const getIcon = type => iconMap[type] ?? '❓';
```

新增类型？直接往 `iconMap` 里添一行键值对即可，告别“再写三行 `else if`”。

---

### 3. 函数版策略模式：把“条件”拆成“策略”

如果每个分支里不是返回值，而是一整段动作逻辑，不妨把动作封装成函数，再按条件调度——这就是轻量级策略模式。

**老写法：**

```js
function handle(code) {
  if (code === 200) renderDashboard();
  else if (code === 401) redirectToLogin();
  else if (code === 403) showForbidden();
  else showError();
}
```

**新写法：**

```js
const strategies = {
  200: renderDashboard,
  401: redirectToLogin,
  403: showForbidden,
  default: showError,
};

const handle = code => (strategies[code] || strategies.default)();
```

函数即策略，职责单一，单元测试也方便。

---

### 4. 短路运算符：让条件“隐形”

只想在特定条件下触发某个副作用，或者给变量兜底一个默认值？`&&`、`||`、`??` 三个符号就能搞定。

- **“满足就执行”**  
  老写法：

  ```js
  if (debug) console.table(data);
  ```
  新写法：

  ```js
  debug && console.table(data);
  ```

- **“兜底默认值”**  
  老写法：

  ```js
  let name;
  if (inputName) name = inputName;
  else name = '游客';
  ```
  新写法（区分假值与 null/undefined）：

  ```js
  const name = inputName ?? '游客';   // 仅 null/undefined 兜底
  // 或
  const name2 = inputName || '游客';  // 任何假值都兜底
  ```

---

### 5. 规则数组：把“阶梯”变“流水线”

当判断涉及区间、权重或更复杂的表达式时，把规则抽象成数组，再让程序按顺序匹配第一条命中的规则——声明式写法优雅又易扩展。

**老写法：**

```js
function shippingFee(weight) {
  if (weight <= 1) return 5;
  else if (weight <= 5) return 10;
  else if (weight <= 10) return 15;
  else return 20;
}
```

**新写法：**

```js
const feeRules = [
  { test: w => w <= 1,  fee: 5  },
  { test: w => w <= 5,  fee: 10 },
  { test: w => w <= 10, fee: 15 },
  { test: () => true,   fee: 20 }, // 兜底
];

const shippingFee = weight => feeRules.find(r => r.test(weight)).fee;
```

新增一个区间？往数组头部插入一条规则即可，无需改动任何判断逻辑。

---

### 小结

| 场景 | 推荐方案 |
|---|---|
| 二元赋值 | 三元运算符 |
| 多值映射 | 对象/Map |
| 多动作分支 | 函数映射（策略） |
| 条件副作用 / 默认值 | 短路运算 |
| 区间/权重判断 | 规则数组 |

`if-else` 本身无罪，但当它开始“横向生长”时，就该考虑换一种更贴合场景的表达方式。选择合适的工具，让代码自己“说话”，未来的你会感谢今天多敲的这一行重构。