## 在开始之前你需要满足以下条件

1. **已安装极简首页** — 扩展需通过 Edge/Chrome 扩展管理页面加载
2. **创建扩展文件夹** — 文件夹内必须包含以下三个文件：
   - `xxx.itnproject` — 元数据（JSON）
   - `xxx.itnprojectcss` — 自定义样式（CSS）
   - `xxx.itnprojectjs` — 逻辑代码（JavaScript）
3. **文件名不限**，后缀匹配即可，三个文件缺一不可，否则校验不通过
4. **安装方式**：打开极简首页 → 设置 → 管理扩展 → 点击"安装" → 选择你的扩展文件夹 → 自动安装

## **一、认识扩展文件结构**

扩展文件夹/

├── xxx.itnproject      — 元数据（JSON）

├── xxx.itnprojectcss   — 自定义样式（CSS）

└── xxx.itnprojectjs    — 逻辑代码（JavaScript）

文件名自取，后缀匹配即可。三个文件缺一不可，否则校验不通过。

## **二、.itnproject 元数据**

以下是演示示例：

```text
{
  "name": "test按钮", // 扩展名
  "version": "1.0.0", // 扩展版本号
  "author": "itnproject", // 扩展作者
  "description": "test按钮出现在首页右下角", // 扩展介绍
  "type": "plugin" // 类型
}
```

| **字段** | **类型** | **必填** | **说明** |
| :-- | :-- | :-- | :-- |
| `name` | string | 否 | 扩展名称，缺省用文件夹名 |
| `version` | string | 否 | 版本号，缺省 `1.0.0` |
| `author` | string | 否 | 作者，缺省 `未知` |
| `description` | string | 否 | 描述，缺省空 |
| `type` | string | 否 | `"theme"` 或 `"plugin"`，缺省 `"plugin"` |

> JSON 解析失败不会中断安装，会回退到默认值。

## **三、.itnprojectjs 运行环境**

JS 代码会被包装在一个函数作用域中执行，所有 API 方法以局部变量形式注入，**直接调用方法名即可，无需任何前缀**。

```text
// 正确格式
showToast('你好');
const btn = createButton('点击', { variant: 'primary' });

// 错误格式 不需要 window.xxx 或任何前缀
window.xxx.showToast('你好');
```

## **四、API 方法详解**

### **4.1 showToast(msg)**

弹出提示消息，与首页设置面板的 toast 一致。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `msg` | string | 提示文本 |

```js
showToast('扩展已加载');
```

---

### **4.2 getSetting(key, defaultValue)**

读取用户设置。设置数据与首页共享，跨扩展可读写。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `key` | string | 设置键名 |
| `defaultValue` | any | 键不存在时的默认值 |

**返回值：** 存储的值，或 `defaultValue`。

```js
const bg = getSetting('customBackground', '');
const dark = getSetting('selectedTheme', 'default') === 'dark';
```

---

### **4.3 setSetting(key, value)**

写入用户设置，会持久化到本地存储，所有扩展共享。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `key` | string | 设置键名 |
| `value` | any | 要存储的值 |

```js
setSetting('myExtLastRun', Date.now());
```

---

### **4.4 addStyle(css)**

注入自定义 CSS 到页面 `<head>`。自动标记 `data-ext-css` 属性，卸载扩展时会被清理。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `css` | string | CSS 样式文本 |

**返回值：** 创建的 `<style>` 元素。

```js
addStyle(`
  .my-ext-widget {
    position: fixed;
    bottom: 100px;
    right: 20px;
    z-index: 9999;
  }
`);
```

---

### **4.5 addElement(html)**

向页面 `<body>` 注入 HTML 片段。自动标记 `data-ext-id` 属性，卸载扩展时会被清理。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `html` | string | HTML 字符串 |

**返回值：** 创建的容器 `<div>` 元素。

```js
const container = addElement(`
  <div class="my-ext-panel">
    <h3>Hello</h3>
  </div>
`);
```

---

### **4.6 createButton(text, config)**

创建符合首页 UI 风格的按钮（`.btn`）。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `text` | string | 按钮文字 |
| `config` | object | 配置对象 |

**config 字段：**

| **字段** | **类型** | **默认** | **说明** |
| :-- | :-- | :-- | :-- |
| `variant` | string | 无 | `"primary"` 紫色主按钮，`"secondary"` 灰色次按钮 |
| `onClick` | function | 无 | 点击回调 |
| `id` | string | 无 | 元素 ID |
| `style` | object | 无 | 内联样式对象 |

**返回值：** 创建的 `<button>` 元素。

```js
const btn = createButton('保存', {
    variant: 'primary',
    onClick: function() {
        showToast('已保存');
    }
});
document.body.appendChild(btn);

const cancel = createButton('取消', {
    variant: 'secondary',
    onClick: function() {
        panel.remove();
    }
});
```

---

### **4.7 createToggle(config)**

创建符合首页 UI 风格的开关（`.toggle-switch`）。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `config` | object | 配置对象 |

**config 字段：**

| **字段** | **类型** | **默认** | **说明** |
| :-- | :-- | :-- | :-- |
| `checked` | boolean | false | 初始是否开启 |
| `label` | string | 无 | 开关右侧文字 |
| `onChange` | function | 无 | 切换回调，参数为当前状态 `boolean` |

**返回值：** 创建的 `<label>` 元素（内含 `<input type="checkbox">` \+ `.toggle-slider`）。

```js
const toggle = createToggle({
    checked: true,
    label: '启用樱花效果',
    onChange: function(on) {
        showToast(on ? '已开启' : '已关闭');
        setSetting('sakuraEnabled', on ? '1' : '0');
    }
});
```

---

### **4.8 createInput(placeholder, config)**

创建符合首页 UI 风格的输入框（`.form-group` \> `input`）。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `placeholder` | string | 占位文本 |
| `config` | object | 配置对象 |

**config 字段：**

| **字段** | **类型** | **默认** | **说明** |
| :-- | :-- | :-- | :-- |
| `type` | string | `"text"` | input type |
| `value` | string | 无 | 初始值 |
| `id` | string | 无 | 元素 ID |
| `onChange` | function | 无 | 输入回调 |

**返回值：** 创建的 `<div class="form-group">` 包裹元素。

```js
const input = createInput('输入城市名...', {
    value: '北京',
    onChange: function() {
        const val = input.querySelector('input').value;
        showToast('输入: ' + val);
    }
});

```

### **4.9 createCard(config)**

创建符合UI的卡片（`.extension-list-item`）。

| **参数** | **类型** | **说明** |
| :-- | :-- | :-- |
| `config` | object | 配置对象 |

**config 字段：**

| **字段** | **类型** | **默认** | **说明** |
| :-- | :-- | :-- | :-- |
| `title` | string | 无 | 卡片标题（`.extension-name` 样式） |
| `content` | string\|Element | 无 | 卡片内容，字符串或 DOM 元素 |
| `id` | string | 无 | 元素 ID |

**返回值：** 创建的卡片 `<div>` 元素。

```javascript
const card = createCard({
    title: '天气信息',
    content: '加载中...'
});

const btn = createButton('刷新', { variant: 'primary' });
card.appendChild(btn);
document.body.appendChild(card);
```

### **4.10 getSearchEngine()**

获取当前用户选择的搜索引擎。

**返回值：** `string` — `"bing"` / `"google"` / `"baidu"` 等。

```js
const engine = getSearchEngine();
showToast('当前引擎: ' + engine);
```

### **4.11 isDarkMode()**

获取当前是否暗色模式。

**返回值：** `boolean`。

```js
if (isDarkMode()) {
    addStyle('.my-widget { color: #fff; }');
} else {
    addStyle('.my-widget { color: #333; }');
}
```

### **4.12 getCurrentBackground()**

获取当前自定义背景图片 URL。

**返回值：** `string` — 图片 URL，无背景时为空字符串。

```js
const bg = getCurrentBackground();
if (bg) {
    addStyle('.my-overlay { background: rgba(0,0,0,0.5); }');
}
```

### **4.13 getSearchHistory()**

获取用户搜索历史记录。

**返回值：** `Array<string>`。

```js
const history = getSearchHistory();
showToast('共 ' + history.length + ' 条搜索记录');
```

## **五、完整扩展示例**

<Check>
  这边使用粉色樱花飘落效果主题来示例 \[骗你的，其实只有弹窗好吧\]
</Check>

### **theme.itnproject**

```json
{
  "name": "粉色樱花飘落效果主题",
  "version": "1.0",
  "author": "itnproject",
  "description": "粉色樱花飘落效果",
  "type": "theme"
}
```

### **theme.itnprojectcss**

```css
.sakura-petal {
    position: fixed;
    pointer-events: none;
    z-index: 10000;
    animation: sakura-fall linear forwards;
}
@keyframes sakura-fall {
    0%   { transform: translateY(-10vh) rotate(0deg); opacity: 1; }
    100% { transform: translateY(110vh) rotate(360deg); opacity: 0; }
}
```

### **theme.itnprojectjs**

```js
(function() {
    function spawnPetal() {
        const petal = document.createElement('div');
        petal.className = 'sakura-petal';
        petal.textContent = '🌸';
        petal.style.left = Math.random() * 100 + '%';
        petal.style.fontSize = (12 + Math.random() * 20) + 'px';
        petal.style.animationDuration = (3 + Math.random() * 5) + 's';
        document.body.appendChild(petal);
        setTimeout(function() { petal.remove(); }, 8000);
    }

    setInterval(spawnPetal, 300);
    showToast('已启用粉色樱花飘落效果主题');
})();
```
