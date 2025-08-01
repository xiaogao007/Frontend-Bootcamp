

## 1. 什么是 Proxy？

Proxy 是 ES6 新增的内置构造函数，用于在运行时生成原始对象的“透明代理”。借助 Proxy，我们可以在不改动原对象的前提下，拦截并重新定义属性读取、赋值、函数调用等底层操作。  
语法：

```js
const proxy = new Proxy(target, handler);
```

- `target`：需要代理的原始对象。  
- `handler`：一个普通对象，其属性是一组“陷阱函数”（trap），对应各类底层操作。当代理对象被访问时，引擎先执行对应的陷阱函数，再决定是否继续转发到原始对象。

## 2. 核心陷阱详解

### 2.1 `get` —— 拦截属性读取

```js
const user = { name: 'Alice', age: 25 };

const userProxy = new Proxy(user, {
  get(target, prop) {
    console.log(`读取属性 ${prop}`);
    return prop in target ? target[prop] : 'default';
  }
});

console.log(userProxy.name);     // 读取属性 name → Alice
console.log(userProxy.notExist); // 读取属性 notExist → default
```

### 2.2 `set` —— 拦截属性赋值

```js
const counter = { value: 0 };

const counterProxy = new Proxy(counter, {
  set(target, prop, value) {
    if (prop === 'value' && typeof value !== 'number') {
      console.log('值必须为数字');
      return false; // 阻止赋值
    }
    target[prop] = value;
    return true;
  }
});

counterProxy.value = 10;   // 成功
counterProxy.value = 'x';  // 值必须为数字
```

### 2.3 `apply` —— 拦截函数调用

```js
function add(a, b) { return a + b; }

const addProxy = new Proxy(add, {
  apply(target, thisArg, argList) {
    console.log(`调用 add(${argList.join(', ')})`);
    if (argList.some(v => typeof v !== 'number')) {
      console.log('参数必须为数字');
      return;
    }
    return Reflect.apply(target, thisArg, argList);
  }
});

addProxy(2, 3);    // 调用 add(2, 3) → 5
addProxy(2, '3');  // 调用 add(2, 3) → 参数必须为数字
```

## 3. 生产级应用方案

### 3.1 数据响应式（简化版 Vue3 核心）

```js
const data = { message: 'Hello' };
const reactive = new Proxy(data, {
  get(t, k) {
    // 依赖收集逻辑
    return t[k];
  },
  set(t, k, v) {
    t[k] = v;
    console.log('数据已更新，触发视图重渲染');
    return true;
  }
});

reactive.message = 'Hello Proxy'; // 数据已更新，触发视图重渲染
```

### 3.2 基于角色的权限控制

```js
const userData = {
  name: 'Bob',
  age: 30,
  email: 'bob@example.com',
  password: '***',
  adminOnly: true
};
const isAdmin = false;

const secureData = new Proxy(userData, {
  get(t, k) {
    if (k === 'password' || (k === 'adminOnly' && !isAdmin)) {
      throw new Error('权限不足');
    }
    return t[k];
  },
  set(t, k, v) {
    if (k === 'password' || (k === 'adminOnly' && !isAdmin)) {
      throw new Error('权限不足');
    }
    t[k] = v;
    return true;
  }
});

console.log(secureData.name);      // Bob
// console.log(secureData.password); // 抛出：权限不足
```

### 3.3 无侵入调试与日志

```js
const api = {
  getUser(id) { return { id, name: 'User' + id }; },
  updateUser(user) { /* 更新逻辑 */ }
};

const loggedApi = new Proxy(api, {
  get(target, prop) {
    const fn = target[prop];
    if (typeof fn !== 'function') return fn;
    return new Proxy(fn, {
      apply(fn, thisArg, argList) {
        console.log(`调用 ${prop}(${argList.join(', ')})`);
        const res = Reflect.apply(fn, thisArg, argList);
        console.log(`返回 ${JSON.stringify(res)}`);
        return res;
      }
    });
  }
});

loggedApi.getUser(1);
// 调用 getUser(1)
// 返回 {"id":1,"name":"User1"}
```

## 4. 使用注意点

- **this 绑定**：陷阱函数内部 `this` 默认指向 `handler`，如需原始对象上下文，可使用 `Reflect` 或手动绑定。  
- **性能**：大规模高频读写场景需评估额外开销，可针对冷数据保持裸对象。  
- **嵌套代理**：多次代理同一对象时，拦截链会叠加，调试信息随之变长，需做好逻辑分层。  
- **兼容性**：IE 全系不支持；移动端需检测 `window.Proxy` 或使用 [proxy-polyfill](https://github.com/GoogleChrome/proxy-polyfill)，但部分陷阱（`apply`, `construct`）无法 polyfill。

## 5. 小结

Proxy 提供了统一、标准化的接口，让我们能够在运行时重写对象行为，而无需改动原始代码。它在数据响应式、权限管理、调试跟踪等场景中已成为现代前端框架和工具链的基石。掌握 Proxy 不仅有助于理解 Vue 3、MobX 等库的内部机制，也能在业务中封装出高可维护性的底层能力。建议读者在真实项目中逐步尝试，并结合性能与兼容性要求做合理取舍。