# DOM与BOM深度

## 概述

文档对象模型（DOM）和浏览器对象模型（BOM）是 Web 开发的两大核心体系。DOM 提供了操作文档内容的标准接口，而 BOM 则封装了浏览器本身的功能。深入理解这两个模型及其 API，是构建高效、交互丰富的前端应用的基础。本文将全面介绍 DOM 节点操作、BOM 核心对象、以及现代观察者 API（IntersectionObserver、ResizeObserver、MutationObserver）的实战应用。

## 技术概述与定位

### DOM 的核心概念

DOM（Document Object Model）是将 HTML 或 XML 文档转换为对象结构的标准编程接口。它将文档的每个部分都视为一个节点（Node），形成一个树形结构，称为 DOM 树。这个树形结构是浏览器渲染页面的基础，也是 JavaScript 与页面交互的桥梁。

DOM 的核心价值在于它提供了一种标准化的方式来访问和操作文档内容。无论使用何种编程语言，只要遵循 DOM 标准，就可以实现对文档的增删改查操作。这使得 Web 开发具有了跨平台、跨语言的一致性。

DOM 规范由 W3C（World Wide Web Consortium）制定和维护。从 DOM Level 1 到 DOM Level 4，规范经历了多次重大更新，引入了更多高效的选择器、遍历方法和操作接口。现代浏览器对 DOM Level 4 的支持已经非常完善，开发者可以放心使用这些新特性。

DOM 的性能特点也需要深入理解。每次 DOM 操作都可能触发浏览器的重排（Reflow）或重绘（Repaint），这是前端性能优化的重要考量点。理解何时会触发重排重绘，如何批量操作 DOM，以及如何使用虚拟 DOM 等技术来优化性能，是每个前端开发者必须掌握的技能。

### BOM 的核心概念

BOM（Browser Object Model）是浏览器提供的独立于网页内容之外的对象模型。与 DOM 不同，BOM 不是 W3C 标准的一部分，因此不同浏览器的 BOM 实现可能存在差异。但随着 Web 标准的演进，BOM 中的一些核心对象已经被标准化，如 `window`、`navigator`、`location` 等。

BOM 提供了与浏览器窗口交互的能力，包括窗口控制、浏览器信息获取、历史记录管理、屏幕信息等。这些功能在现代 Web 应用中仍然扮演着重要角色，尽管有些功能已经被新的 Web API 所取代或补充。

`window` 对象是 BOM 的核心，它是浏览器窗口或帧的全局对象。在浏览器环境中，`window` 对象包含了所有全局变量、全局函数和 DOM 文档的入口。理解 `window` 对象的层级结构和各个属性的作用，对于掌握 BOM 至关重要。

### DOM 与 BOM 的关系

DOM 和 BOM 虽然是独立的概念，但在实际应用中紧密协作。DOM 负责文档内容的管理，而 BOM 负责浏览器环境的管理。当我们使用 JavaScript 操作网页时，往往需要同时使用 DOM 和 BOM 的 API。

例如，在响应窗口大小变化时，我们需要使用 BOM 的 `window.resize` 事件来监听变化，然后使用 DOM 的 API 来更新页面的布局。在进行页面导航时，我们使用 BOM 的 `location` 对象来获取或修改 URL，同时 DOM 会根据新的 URL 更新文档内容。

---

## 完整安装与配置

### 开发环境准备

现代 Web 开发通常使用构建工具来管理项目，但理解原生 DOM 和 BOM API 仍然是前端开发的基础。以下是一个标准的前端开发环境配置：

```json
// package.json
{
  "name": "dom-bom-exploration",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint src/**/*.js"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "eslint": "^8.55.0"
  }
}
```

```javascript
// vite.config.js
export default {
  server: {
    port: 3000,
    open: true
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  }
}
```

### 浏览器开发者工具配置

为了更好地学习和调试 DOM 和 BOM，配置好浏览器开发者工具至关重要：

```javascript
// 在控制台中启用详细的 DOM 变更日志
config.set('DOM-breakpoints', true);

// 启用事件监听器断点
config.set('EventListeners-breakpoints', true);

// 设置日志保存
config.set('Preserve log', true);

// 启用网络请求日志
config.set('Network log', true);
```

### 现代开发中的 DOM 操作库

虽然原生 DOM API 功能强大，但在大型应用中，使用封装库可以提高开发效率和代码可维护性：

```javascript
// 使用 jQuery 的 DOM 操作方式（传统方式）
$('#container').append('<div class="item">Content</div>');
$('.item').on('click', handleClick);

// 使用原生 API 的现代方式
document.querySelector('#container').insertAdjacentHTML('beforeend', '<div class="item">Content</div>');
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleClick);
});

// 使用现代 DOM API
const fragment = document.createDocumentFragment();
const container = document.getElementById('container');

[1, 2, 3, 4, 5].forEach(num => {
  const div = document.createElement('div');
  div.className = 'item';
  div.textContent = `Item ${num}`;
  fragment.appendChild(div);
});

container.appendChild(fragment);
```

---

## 核心概念详解

### DOM 节点类型与关系详解

DOM 将 HTML 文档表示为树形结构，包含多种节点类型：

## DOM 节点操作完整 API

### 节点类型与关系

DOM 将 HTML 文档表示为树形结构，包含多种节点类型：

| 节点类型 | nodeType 值 | 说明 |
|----------|-------------|------|
| Element | 1 | HTML 元素 |
| Text | 3 | 文本节点 |
| Comment | 8 | 注释节点 |
| Document | 9 | 整个文档 |
| DocumentType | 10 | DOCTYPE |
| DocumentFragment | 11 | 文档片段 |

```javascript
// 节点类型判断
function identifyNode(node) {
  const typeMap = {
    1: 'Element',
    2: 'Attr',
    3: 'Text',
    8: 'Comment',
    9: 'Document',
    10: 'DocumentType',
    11: 'DocumentFragment'
  };
  
  return typeMap[node.nodeType] || 'Unknown';
}

// 节点关系判断
function analyzeRelations(element) {
  return {
    parent: element.parentElement,
    parentNode: element.parentNode,
    children: [...element.children],
    childNodes: [...element.childNodes],
    firstChild: element.firstChild,
    lastChild: element.lastChild,
    nextSibling: element.nextSibling,
    previousSibling: element.previousSibling,
    nextElementSibling: element.nextElementSibling,
    previousElementSibling: element.previousElementSibling,
    textContent: element.textContent,
    innerHTML: element.innerHTML
  };
}
```

### 创建节点

```javascript
class DOMFactory {
  // 创建元素
  static createElement(tagName, attributes = {}, children = []) {
    const element = document.createElement(tagName);
    
    Object.entries(attributes).forEach(([key, value]) => {
      if (key === 'className') {
        element.className = value;
      } else if (key === 'dataset') {
        Object.entries(value).forEach(([dataKey, dataValue]) => {
          element.dataset[dataKey] = dataValue;
        });
      } else if (key === 'style' && typeof value === 'object') {
        Object.assign(element.style, value);
      } else if (key.startsWith('on') && typeof value === 'function') {
        element.addEventListener(key.slice(2).toLowerCase(), value);
      } else {
        element.setAttribute(key, value);
      }
    });
    
    children.forEach(child => {
      if (typeof child === 'string') {
        element.appendChild(document.createTextNode(child));
      } else if (child instanceof Node) {
        element.appendChild(child);
      }
    });
    
    return element;
  }

  // 创建文档片段
  static createFragment(children = []) {
    const fragment = document.createDocumentFragment();
    children.forEach(child => {
      if (typeof child === 'string') {
        fragment.appendChild(document.createTextNode(child));
      } else if (child instanceof Node) {
        fragment.appendChild(child);
      }
    });
    return fragment;
  }

  // 批量创建元素（高性能）
  static createElements(specs) {
    const fragment = document.createDocumentFragment();
    
    specs.forEach(({ tag, attrs = {}, children = [] }) => {
      const element = this.createElement(tag, attrs, children);
      fragment.appendChild(element);
    });
    
    return fragment;
  }
}

// 使用示例：构建复杂 UI
function buildUserCard(user) {
  const card = DOMFactory.createElement('div', {
    className: 'user-card',
    dataset: { userId: user.id }
  }, [
    DOMFactory.createElement('img', {
      src: user.avatar,
      alt: user.name,
      className: 'avatar',
      style: { width: '64px', height: '64px', borderRadius: '50%' }
    }),
    DOMFactory.createElement('div', { className: 'info' }, [
      DOMFactory.createElement('h3', {}, [user.name]),
      DOMFactory.createElement('p', { className: 'bio' }, [user.bio]),
      DOMFactory.createElement('button', {
        className: 'btn-follow',
        onClick: () => followUser(user.id)
      }, ['关注'])
    ])
  ]);
  
  return card;
}
```

### 查询节点

```javascript
class DOMQuery {
  // 单元素查询
  static $(selector, context = document) {
    return context.querySelector(selector);
  }

  // 多元素查询
  static $$(selector, context = document) {
    return [...context.querySelectorAll(selector)];
  }

  // 按属性查询
  static byAttribute(attribute, value = null, context = document) {
    if (value === null) {
      return this.$$(`[${attribute}]`, context);
    }
    return this.$$(`[${attribute}="${value}"]`, context);
  }

  // 按文本内容查询
  static byText(text, context = document) {
    return this.$$('*', context).filter(
      el => el.textContent.toLowerCase().includes(text.toLowerCase())
    );
  }

  // 按节点类型查询
  static byType(tagName, context = document) {
    return this.$$(tagName, context);
  }

  // 相对查询
  static relative(element, direction) {
    const directions = {
      'parent': el => el.parentElement,
      'closest-parent': el => el.closest('*'),
      'first-child': el => el.firstElementChild,
      'last-child': el => el.lastElementChild,
      'next': el => el.nextElementSibling,
      'prev': el => el.previousElementSibling,
      'children': el => [...el.children]
    };
    
    if (Array.isArray(direction)) {
      return direction.map(dir => directions[dir]?.(element)).filter(Boolean);
    }
    
    return directions[direction]?.(element);
  }
}

// 高级选择器
function advancedQuery() {
  // 获取所有带有特定类名的直接子元素
  const directChildren = document.querySelectorAll(':scope > .child');
  
  // 获取非隐藏元素
  const visibleElements = document.querySelectorAll(':not([hidden])');
  
  // 获取可交互元素
  const interactiveElements = document.querySelectorAll(
    'button:not([disabled]), a:not([href="#"]), input:not([disabled])'
  );
  
  // 获取特定属性格式的元素
  const dataElements = document.querySelectorAll('[data-id^="user-"]');
}
```

### 节点操作

```javascript
class DOMManipulator {
  // 插入节点
  static insert(element, target, position = 'beforeend') {
    const positions = {
      'beforebegin': 'beforebegin',
      'afterbegin': 'afterbegin',
      'beforeend': 'beforeend',
      'afterend': 'afterend'
    };
    target.insertAdjacentElement(positions[position], element);
  }

  // 批量插入 HTML
  static insertHTML(target, html, position = 'beforeend') {
    target.insertAdjacentHTML(position, html);
  }

  // 移动节点
  static move(element, newParent, position = 'beforeend') {
    if (element.parentElement === newParent) return;
    this.insert(element, newParent, position);
  }

  // 替换节点
  static replace(oldElement, newElement) {
    oldElement.replaceWith(newElement);
  }

  // 删除节点
  static remove(element) {
    element.remove();
  }

  // 克隆节点
  static clone(element, deep = true) {
    return element.cloneNode(deep);
  }

  // 批量操作（文档片段优化）
  static batchInsert(parent, elements) {
    const fragment = document.createDocumentFragment();
    elements.forEach(el => fragment.appendChild(el));
    parent.appendChild(fragment);
  }

  // 动画化插入
  static animateInsert(element, target, position = 'beforeend', animation = 'fadeIn') {
    const animations = {
      fadeIn: `
        @keyframes fadeIn {
          from { opacity: 0; transform: translateY(-10px); }
          to { opacity: 1; transform: translateY(0); }
        }
      `,
      slideIn: `
        @keyframes slideIn {
          from { max-height: 0; opacity: 0; }
          to { max-height: 500px; opacity: 1; }
        }
      `
    };
    
    const styleId = 'dom-manipulator-styles';
    if (!document.getElementById(styleId)) {
      const style = document.createElement('style');
      style.id = styleId;
      style.textContent = Object.values(animations).join('\n');
      document.head.appendChild(style);
    }
    
    element.style.animation = `${animation} 0.3s ease-out`;
    this.insert(element, target, position);
  }
}

// 拖拽排序实现
class SortableList {
  constructor(container, options = {}) {
    this.container = container;
    this.items = [...container.children];
    this.draggedItem = null;
    this.draggedIndex = null;
    this.onSort = options.onSort || (() => {});
    
    this.init();
  }

  init() {
    this.container.addEventListener('dragstart', this.handleDragStart.bind(this));
    this.container.addEventListener('dragover', this.handleDragOver.bind(this));
    this.container.addEventListener('dragend', this.handleDragEnd.bind(this));
    this.container.addEventListener('drop', this.handleDrop.bind(this));
    
    this.items.forEach((item, index) => {
      item.draggable = true;
      item.dataset.index = index;
    });
  }

  handleDragStart(e) {
    this.draggedItem = e.target;
    this.draggedIndex = parseInt(e.target.dataset.index);
    e.target.classList.add('dragging');
    e.dataTransfer.effectAllowed = 'move';
  }

  handleDragOver(e) {
    e.preventDefault();
    e.dataTransfer.dropEffect = 'move';
    
    const afterElement = this.getDragAfterElement(e.clientY);
    if (afterElement === null) {
      this.container.appendChild(this.draggedItem);
    } else {
      this.container.insertBefore(this.draggedItem, afterElement);
    }
  }

  handleDragEnd(e) {
    e.target.classList.remove('dragging');
    const newIndex = [...this.container.children].indexOf(this.draggedItem);
    this.onSort({ oldIndex: this.draggedIndex, newIndex });
  }

  handleDrop(e) {
    e.preventDefault();
  }

  getDragAfterElement(y) {
    const draggableElements = [...this.container.children]
      .filter(el => el !== this.draggedItem);
    
    return draggableElements.reduce((closest, child) => {
      const box = child.getBoundingClientRect();
      const offset = y - box.top - box.height / 2;
      
      if (offset < 0 && offset > closest.offset) {
        return { offset, element: child };
      }
      return closest;
    }, { offset: Number.NEGATIVE_INFINITY }).element;
  }
}
```

## BOM 核心对象

### window 对象

```javascript
// window 属性与方法
class WindowAPI {
  // 尺寸相关
  static getViewportSize() {
    return {
      width: window.innerWidth,
      height: window.innerHeight
    };
  }

  static getDocumentSize() {
    return {
      width: Math.max(
        document.body.scrollWidth,
        document.documentElement.scrollWidth,
        document.body.offsetWidth,
        document.documentElement.offsetWidth,
        document.documentElement.clientWidth
      ),
      height: Math.max(
        document.body.scrollHeight,
        document.documentElement.scrollHeight,
        document.body.offsetHeight,
        document.documentElement.offsetHeight,
        document.documentElement.clientHeight
      )
    };
  }

  // 滚动控制
  static scrollTo(x, y, behavior = 'smooth') {
    window.scrollTo({ top: y, left: x, behavior });
  }

  static scrollIntoView(element, options = {}) {
    element.scrollIntoView({
      behavior: options.behavior || 'smooth',
      block: options.block || 'center',
      inline: options.inline || 'nearest'
    });
  }

  // 防抖/节流
  static debounce(fn, delay) {
    let timeoutId;
    return function(...args) {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => fn.apply(this, args), delay);
    };
  }

  static throttle(fn, limit) {
    let inThrottle;
    return function(...args) {
      if (!inThrottle) {
        fn.apply(this, args);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    };
  }

  // 全屏 API
  static async toggleFullscreen(element = document.documentElement) {
    if (!document.fullscreenElement) {
      await element.requestFullscreen();
    } else {
      await document.exitFullscreen();
    }
  }

  // 剪贴板
  static async copyToClipboard(text) {
    try {
      await navigator.clipboard.writeText(text);
      return true;
    } catch (err) {
      // fallback
      const textarea = document.createElement('textarea');
      textarea.value = text;
      textarea.style.position = 'fixed';
      textarea.style.opacity = '0';
      document.body.appendChild(textarea);
      textarea.select();
      const success = document.execCommand('copy');
      document.body.removeChild(textarea);
      return success;
    }
  }
}

// 窗口事件管理
class WindowEventManager {
  constructor() {
    this.handlers = new Map();
  }

  on(event, handler, options = {}) {
    window.addEventListener(event, handler, options);
    
    if (!this.handlers.has(event)) {
      this.handlers.set(event, []);
    }
    this.handlers.get(event).push({ handler, options });
  }

  off(event, handler) {
    const eventHandlers = this.handlers.get(event);
    if (eventHandlers) {
      eventHandlers.forEach(({ handler: h, options }) => {
        window.removeEventListener(event, h, options);
      });
      this.handlers.delete(event);
    }
  }

  once(event, handler) {
    this.on(event, handler, { once: true });
  }
}
```

### navigator 对象

```javascript
class NavigatorAPI {
  // 用户代理信息
  static getUserAgent() {
    return navigator.userAgent;
  }

  // 平台信息
  static getPlatform() {
    return {
      platform: navigator.platform,
      language: navigator.language,
      languages: navigator.languages,
      onLine: navigator.onLine
    };
  }

  // 硬件信息
  static getHardwareInfo() {
    return {
      hardwareConcurrency: navigator.hardwareConcurrency,
      deviceMemory: navigator.deviceMemory,
      maxTouchPoints: navigator.maxTouchPoints
    };
  }

  // 电池 API
  static async getBatteryInfo() {
    if ('getBattery' in navigator) {
      const battery = await navigator.getBattery();
      return {
        charging: battery.charging,
        level: battery.level * 100,
        chargingTime: battery.chargingTime,
        dischargingTime: battery.dischargingTime
      };
    }
    return null;
  }

  // 网络信息
  static getNetworkInfo() {
    const connection = navigator.connection || 
                       navigator.mozConnection || 
                       navigator.webkitConnection;
    
    if (connection) {
      return {
        effectiveType: connection.effectiveType,
        downlink: connection.downlink,
        rtt: connection.rtt,
        saveData: connection.saveData
      };
    }
    return null;
  }

  // 存储 API
  static async isStoragePersisted() {
    if (navigator.storage && navigator.storage.persisted) {
      return await navigator.storage.persisted();
    }
    return false;
  }

  static async persistStorage() {
    if (navigator.storage && navigator.storage.persist) {
      return await navigator.storage.persist();
    }
    return false;
  }

  // 权限 API
  static async checkPermission(permissionName) {
    try {
      const result = await navigator.permissions.query({
        name: permissionName
      });
      return {
        state: result.state,
        onchange: () => {
          result.addEventListener('change', () => {
            console.log(`Permission ${permissionName} changed to ${result.state}`);
          });
        }
      };
    } catch (err) {
      console.error('Permission query failed:', err);
      return null;
    }
  }

  // 设备方向
  static requestDeviceOrientation(callback) {
    if ('DeviceOrientationEvent' in window) {
      window.addEventListener('deviceorientation', callback);
      return () => {
        window.removeEventListener('deviceorientation', callback);
      };
    }
    return null;
  }
}
```

### history 对象

```javascript
class HistoryAPI {
  // 导航
  static pushState(data, title, url) {
    history.pushState(data, title, url);
  }

  static replaceState(data, title, url) {
    history.replaceState(data, title, url);
  }

  static go(delta) {
    history.go(delta);
  }

  static back() {
    history.back();
  }

  static forward() {
    history.forward();
  }

  // 状态管理
  static getState() {
    return history.state;
  }

  // 监听 popstate（浏览器前进后退）
  static onPopState(callback) {
    window.addEventListener('popstate', callback);
    return () => window.removeEventListener('popstate', callback);
  }
}

// SPA 路由实现
class Router {
  constructor() {
    this.routes = new Map();
    this.currentRoute = null;
    
    HistoryAPI.onPopState(this.handleRouteChange.bind(this));
  }

  addRoute(path, handler) {
    this.routes.set(path, handler);
  }

  navigate(path, options = {}) {
    const data = options.data || {};
    const title = options.title || '';
    
    if (options.replace) {
      HistoryAPI.replaceState(data, title, path);
    } else {
      HistoryAPI.pushState(data, title, path);
    }
    
    this.currentRoute = path;
    this.routes.get(path)?.();
  }

  handleRouteChange() {
    const path = window.location.pathname;
    this.currentRoute = path;
    
    const handler = this.routes.get(path);
    if (handler) {
      handler();
    } else {
      this.routes.get('*')?.();
    }
  }

  init() {
    this.handleRouteChange();
  }
}

// 使用示例
const router = new Router();

router.addRoute('/', () => {
  document.getElementById('app').innerHTML = '<h1>首页</h1>';
});

router.addRoute('/about', () => {
  document.getElementById('app').innerHTML = '<h1>关于我们</h1>';
});

router.addRoute('/user/:id', () => {
  const id = window.location.pathname.split('/').pop();
  document.getElementById('app').innerHTML = `<h1>用户: ${id}</h1>`;
});

router.addRoute('*', () => {
  document.getElementById('app').innerHTML = '<h1>404</h1>';
});

router.init();
```

### location 对象

```javascript
class LocationAPI {
  // URL 解析
  static parse(url = window.location.href) {
    const { protocol, hostname, port, pathname, search, hash, href } = new URL(url);
    return {
      protocol,
      hostname,
      port,
      pathname,
      search,
      hash,
      href,
      origin: `${protocol}//${hostname}${port ? ':' + port : ''}`
    };
  }

  // 查询参数
  static getQueryParams(url = window.location.href) {
    const urlObj = new URL(url);
    return Object.fromEntries(urlObj.searchParams);
  }

  static setQueryParams(params) {
    const url = new URL(window.location.href);
    Object.entries(params).forEach(([key, value]) => {
      if (value === null || value === undefined) {
        url.searchParams.delete(key);
      } else {
        url.searchParams.set(key, value);
      }
    });
    window.location.href = url.toString();
  }

  // URL 修改
  static reload(force = false) {
    window.location.reload(force);
  }

  static assign(url) {
    window.location.assign(url);
  }

  static replace(url) {
    window.location.replace(url);
  }

  // Hash 路由
  static getHash() {
    return window.location.hash.slice(1);
  }

  static setHash(hash) {
    window.location.hash = hash;
  }

  static onHashChange(callback) {
    window.addEventListener('hashchange', (e) => {
      callback({
        oldURL: e.oldURL,
        newURL: e.newURL,
        hash: window.location.hash.slice(1)
      });
    });
  }
}
```

## Intersection Observer 实战

### 基础用法

```javascript
class LazyLoader {
  constructor(options = {}) {
    this.options = {
      root: options.root || null,
      rootMargin: options.rootMargin || '0px',
      threshold: options.threshold || 0
    };
    this.observer = null;
    this.callbacks = new Map();
  }

  observe(element, callback) {
    if (!this.observer) {
      this.observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          const callback = this.callbacks.get(entry.target);
          if (callback) {
            callback({
              isIntersecting: entry.isIntersecting,
              intersectionRatio: entry.intersectionRatio,
              boundingClientRect: entry.boundingClientRect,
              target: entry.target
            });
          }
        });
      }, this.options);
    }

    this.callbacks.set(element, callback);
    this.observer.observe(element);
    
    return () => {
      this.callbacks.delete(element);
      this.observer.unobserve(element);
    };
  }

  disconnect() {
    this.observer?.disconnect();
    this.callbacks.clear();
  }
}

// 图片懒加载
function lazyLoadImages() {
  const loader = new LazyLoader({
    rootMargin: '100px' // 提前 100px 开始加载
  });

  document.querySelectorAll('img[data-src]').forEach(img => {
    loader.observe(img, (info) => {
      if (info.isIntersecting) {
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
        img.classList.add('loaded');
      }
    });
  });
}
```

### 无限滚动

```javascript
class InfiniteScroll {
  constructor(container, loadMore, options = {}) {
    this.container = container;
    this.loadMore = loadMore;
    this.isLoading = false;
    this.hasMore = true;
    this.page = 1;
    
    this.options = {
      threshold: options.threshold || 0,
      rootMargin: options.rootMargin || '200px'
    };
    
    this.init();
  }

  init() {
    const sentinel = document.createElement('div');
    sentinel.className = 'scroll-sentinel';
    sentinel.style.height = '1px';
    this.container.appendChild(sentinel);

    const observer = new IntersectionObserver((entries) => {
      const entry = entries[0];
      if (entry.isIntersecting && !this.isLoading && this.hasMore) {
        this.loadData();
      }
    }, this.options);

    observer.observe(sentinel);
    this.observer = observer;
  }

  async loadData() {
    this.isLoading = true;
    
    try {
      const items = await this.loadMore(this.page);
      
      if (items.length === 0) {
        this.hasMore = false;
        this.showEndMessage();
      } else {
        this.appendItems(items);
        this.page++;
      }
    } catch (error) {
      this.showError(error);
    } finally {
      this.isLoading = false;
    }
  }

  appendItems(items) {
    const fragment = document.createDocumentFragment();
    
    items.forEach(item => {
      const element = this.createItemElement(item);
      fragment.appendChild(element);
    });

    const sentinel = this.container.querySelector('.scroll-sentinel');
    this.container.insertBefore(fragment, sentinel);
  }

  createItemElement(item) {
    const div = document.createElement('div');
    div.className = 'list-item';
    div.innerHTML = `
      <h3>${item.title}</h3>
      <p>${item.description}</p>
    `;
    return div;
  }

  showEndMessage() {
    const endMessage = document.createElement('div');
    endMessage.className = 'end-message';
    endMessage.textContent = '没有更多数据了';
    this.container.appendChild(endMessage);
  }

  showError(error) {
    console.error('Load error:', error);
  }

  reset() {
    this.page = 1;
    this.hasMore = true;
    this.isLoading = false;
  }

  destroy() {
    this.observer.disconnect();
  }
}

// 使用示例
const infiniteScroll = new InfiniteScroll(
  document.getElementById('list-container'),
  async (page) => {
    const response = await fetch(`/api/items?page=${page}&limit=20`);
    return response.json();
  },
  { rootMargin: '300px' }
);
```

## ResizeObserver、MutationObserver

### ResizeObserver

```javascript
class ResponsiveManager {
  constructor() {
    this.observers = new Map();
    this.breakpoints = {
      sm: 640,
      md: 768,
      lg: 1024,
      xl: 1280,
      '2xl': 1536
    };
  }

  observe(element, callback) {
    const observer = new ResizeObserver((entries) => {
      entries.forEach(entry => {
        const { width, height } = entry.contentRect;
        const breakpoints = this.calculateBreakpoints(width);
        
        callback({
          width,
          height,
          breakpoints,
          aspectRatio: width / height,
          entry
        });
      });
    });

    observer.observe(element);
    this.observers.set(element, observer);
    
    return () => {
      observer.disconnect();
      this.observers.delete(element);
    };
  }

  calculateBreakpoints(width) {
    return Object.entries(this.breakpoints).reduce((acc, [name, value]) => {
      acc[name] = width >= value;
      return acc;
    }, {});
  }

  disconnectAll() {
    this.observers.forEach(observer => observer.disconnect());
    this.observers.clear();
  }
}

// 图表自适应
function observeChartResize(chartElement, chartInstance) {
  const manager = new ResponsiveManager();
  
  manager.observe(chartElement, ({ width, height }) => {
    chartInstance.resize(width, height);
  });
}
```

### MutationObserver

```javascript
class DOMWatcher {
  constructor() {
    this.observer = null;
    this.handlers = [];
  }

  watch(target, options = {}) {
    const defaultOptions = {
      childList: options.childList ?? true,
      subtree: options.subtree ?? true,
      attributes: options.attributes ?? false,
      attributeOldValue: options.attributeOldValue ?? false,
      characterData: options.characterData ?? false,
      characterDataOldValue: options.characterDataOldValue ?? false
    };

    this.observer = new MutationObserver((mutations) => {
      mutations.forEach(mutation => {
        this.handlers.forEach(handler => {
          handler({
            type: mutation.type,
            target: mutation.target,
            addedNodes: [...mutation.addedNodes],
            removedNodes: [...mutation.removedNodes],
            oldValue: mutation.oldValue,
            attributeName: mutation.attributeName
          });
        });
      });
    });

    this.observer.observe(target, defaultOptions);
    
    return () => this.observer.disconnect();
  }

  onMutation(handler) {
    this.handlers.push(handler);
    return () => {
      const index = this.handlers.indexOf(handler);
      if (index > -1) this.handlers.splice(index, 1);
    };
  }

  disconnect() {
    this.observer?.disconnect();
    this.handlers = [];
  }
}

// 实战：检测元素内容变化
function watchContentChanges(element, callback) {
  const watcher = new DOMWatcher();
  
  watcher.watch(element, { characterData: true, subtree: true });
  watcher.onMutation(({ type, target }) => {
    if (type === 'characterData') {
      callback({
        newValue: target.textContent,
        element: target.parentElement
      });
    }
  });
  
  return () => watcher.disconnect();
}

// 实战：检测 DOM 结构变化
function watchDOMStructure(container, options = {}) {
  const watcher = new DOMWatcher();
  
  watcher.watch(container, {
    childList: true,
    subtree: true
  });
  
  watcher.onMutation(({ addedNodes, removedNodes }) => {
    addedNodes.forEach(node => {
      if (node.nodeType === 1) { // Element node
        options.onAdded?.(node);
      }
    });
    
    removedNodes.forEach(node => {
      if (node.nodeType === 1) {
        options.onRemoved?.(node);
      }
    });
  });
  
  return () => watcher.disconnect();
}
```

---

## 常用命令与操作

### 节点遍历命令

DOM 提供了丰富的遍历方法，用于在节点树中移动：

```javascript
class DOMTraversal {
  // 向上遍历
  static getAncestors(element, maxLevels = Infinity) {
    const ancestors = [];
    let current = element.parentElement;
    let levels = 0;

    while (current && levels < maxLevels) {
      ancestors.push(current);
      current = current.parentElement;
      levels++;
    }

    return ancestors;
  }

  // 查找最近的匹配祖先
  static closest(element, selector) {
    return element.closest(selector);
  }

  // 向下遍历
  static getDescendants(element, selector = '*') {
    return element.querySelectorAll(selector);
  }

  // 获取所有兄弟节点
  static getSiblings(element) {
    const parent = element.parentElement;
    return [...parent.children].filter(child => child !== element);
  }

  // 获取前后兄弟元素
  static getPrevSibling(element) {
    return element.previousElementSibling;
  }

  static getNextSibling(element) {
    return element.nextElementSibling;
  }

  // 遍历树（DFS）
  static traverseDFS(element, callback) {
    callback(element);
    for (const child of element.children) {
      this.traverseDFS(child, callback);
    }
  }

  // 遍历树（BFS）
  static traverseBFS(element, callback) {
    const queue = [element];

    while (queue.length > 0) {
      const node = queue.shift();
      callback(node);
      queue.push(...node.children);
    }
  }
}
```

### 批量操作命令

```javascript
class BatchDOMOperations {
  // 批量创建元素
  static createElements(specs) {
    const fragment = document.createDocumentFragment();

    specs.forEach(spec => {
      const { tag, attrs = {}, children = [], text } = spec;
      const element = document.createElement(tag);

      Object.entries(attrs).forEach(([key, value]) => {
        if (key === 'className') {
          element.className = value;
        } else if (key === 'dataset') {
          Object.entries(value).forEach(([dataKey, dataValue]) => {
            element.dataset[dataKey] = dataValue;
          });
        } else if (key === 'style' && typeof value === 'object') {
          Object.assign(element.style, value);
        } else if (key.startsWith('on') && typeof value === 'function') {
          element.addEventListener(key.slice(2).toLowerCase(), value);
        } else {
          element.setAttribute(key, value);
        }
      });

      if (text) {
        element.textContent = text;
      }

      children.forEach(child => {
        if (typeof child === 'string') {
          element.appendChild(document.createTextNode(child));
        } else if (child instanceof Node) {
          element.appendChild(child);
        }
      });

      fragment.appendChild(element);
    });

    return fragment;
  }

  // 批量替换元素
  static replaceAll(oldElements, newElements) {
    const parent = oldElements[0]?.parentElement;
    if (!parent) return;

    const fragment = document.createDocumentFragment();
    newElements.forEach(el => fragment.appendChild(el));

    oldElements.forEach(el => el.remove());
    parent.appendChild(fragment);
  }

  // 批量移除满足条件的元素
  static removeWhere(container, predicate) {
    [...container.children]
      .filter(predicate)
      .forEach(el => el.remove());
  }
}
```

### 事件操作命令

```javascript
class EventOperations {
  // 事件委托
  static delegate(container, selector, eventType, handler) {
    container.addEventListener(eventType, event => {
      const target = event.target.closest(selector);
      if (target && container.contains(target)) {
        handler.call(target, event);
      }
    });
  }

  // 防抖事件
  static debounceEvent(element, eventType, handler, delay) {
    let timeoutId;

    element.addEventListener(eventType, event => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => {
        handler(event);
      }, delay);
    });

    return () => clearTimeout(timeoutId);
  }

  // 节流事件
  static throttleEvent(element, eventType, handler, limit) {
    let inThrottle = false;

    element.addEventListener(eventType, event => {
      if (!inThrottle) {
        handler(event);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    });
  }

  // 一次性事件
  static once(element, eventType, handler) {
    element.addEventListener(eventType, handler, { once: true });
  }

  // 自定义事件
  static emit(element, eventName, detail = {}) {
    const event = new CustomEvent(eventName, {
      bubbles: true,
      cancelable: true,
      detail
    });
    element.dispatchEvent(event);
  }
}
```

---

## 高级配置与技巧

### 虚拟滚动实现

对于大量数据的列表，虚拟滚动可以显著提升性能：

```javascript
class VirtualScroller {
  constructor(container, options = {}) {
    this.container = container;
    this.itemHeight = options.itemHeight || 50;
    this.bufferSize = options.bufferSize || 5;
    this.items = options.items || [];
    this.renderItem = options.renderItem || (item => document.createTextNode(item));

    this.scrollTop = 0;
    this.viewportHeight = 0;
    this.totalHeight = this.items.length * this.itemHeight;

    this.init();
  }

  init() {
    this.content = document.createElement('div');
    this.content.className = 'virtual-scroll-content';
    this.content.style.height = `${this.totalHeight}px`;
    this.content.style.position = 'relative';

    this.viewport = document.createElement('div');
    this.viewport.className = 'virtual-scroll-viewport';
    this.viewport.style.overflow = 'auto';
    this.viewport.appendChild(this.content);

    this.container.appendChild(this.viewport);

    this.viewport.addEventListener('scroll', this.handleScroll.bind(this));

    this.updateViewport();
    this.render();
  }

  handleScroll() {
    this.scrollTop = this.viewport.scrollTop;
    this.render();
  }

  updateViewport() {
    this.viewportHeight = this.viewport.clientHeight;
  }

  getVisibleRange() {
    const startIndex = Math.max(0,
      Math.floor(this.scrollTop / this.itemHeight) - this.bufferSize
    );

    const visibleCount = Math.ceil(this.viewportHeight / this.itemHeight);
    const endIndex = Math.min(
      this.items.length,
      startIndex + visibleCount + this.bufferSize * 2
    );

    return { startIndex, endIndex };
  }

  render() {
    const { startIndex, endIndex } = this.getVisibleRange();

    // 清除现有内容
    const existingItems = this.content.querySelectorAll('.virtual-scroll-item');
    existingItems.forEach(item => item.remove());

    // 渲染可见项
    for (let i = startIndex; i < endIndex; i++) {
      const item = this.items[i];
      const element = this.renderItem(item);

      element.className = 'virtual-scroll-item';
      element.style.position = 'absolute';
      element.style.top = `${i * this.itemHeight}px`;
      element.style.left = '0';
      element.style.right = '0';
      element.style.height = `${this.itemHeight}px`;

      this.content.appendChild(element);
    }
  }

  setItems(items) {
    this.items = items;
    this.totalHeight = items.length * this.itemHeight;
    this.content.style.height = `${this.totalHeight}px`;
    this.render();
  }

  scrollToIndex(index) {
    this.viewport.scrollTop = index * this.itemHeight;
  }

  destroy() {
    this.viewport.remove();
  }
}

// 使用示例
const scroller = new VirtualScroller(
  document.getElementById('list-container'),
  {
    itemHeight: 60,
    bufferSize: 5,
    items: generateLargeDataset(10000),
    renderItem: (item) => {
      const div = document.createElement('div');
      div.textContent = item.name;
      div.onclick = () => console.log(item.id);
      return div;
    }
  }
);
```

### DOM 性能监控配置

```javascript
class DOMPerformanceMonitor {
  constructor() {
    this.metrics = {
      reflowCount: 0,
      repaintCount: 0,
      mutations: [],
      eventHandlers: new Map()
    };

    this.setupMonitoring();
  }

  setupMonitoring() {
    // 监控重排
    const originalGetBCR = Element.prototype.getBoundingClientRect;
    Element.prototype.getBoundingClientRect = function() {
      DOMPerformanceMonitor.instance?.recordReflow();
      return originalGetBCR.apply(this, arguments);
    };

    // 监控样式计算
    const originalCS = window.getComputedStyle;
    window.getComputedStyle = function(element, pseudoElt) {
      DOMPerformanceMonitor.instance?.recordStyleCalculation();
      return originalCS.apply(this, arguments);
    };

    // 监控 MutationObserver
    this.setupMutationObserver();
  }

  setupMutationObserver() {
    const observer = new MutationObserver(mutations => {
      mutations.forEach(mutation => {
        this.metrics.mutations.push({
          type: mutation.type,
          target: mutation.target.nodeName,
          addedNodes: mutation.addedNodes.length,
          removedNodes: mutation.removedNodes.length,
          timestamp: performance.now()
        });
      });
    });

    observer.observe(document.body, {
      childList: true,
      subtree: true,
      attributes: true,
      attributeOldValue: true,
      characterData: true,
      characterDataOldValue: true
    });
  }

  recordReflow() {
    this.metrics.reflowCount++;
  }

  recordStyleCalculation() {
    this.metrics.repaintCount++;
  }

  trackEventHandler(element, eventType) {
    const key = `${element.nodeName}-${eventType}`;
    const count = this.metrics.eventHandlers.get(key) || 0;
    this.metrics.eventHandlers.set(key, count + 1);
  }

  getReport() {
    return {
      ...this.metrics,
      summary: {
        totalReflows: this.metrics.reflowCount,
        totalRepaints: this.metrics.repaintCount,
        totalMutations: this.metrics.mutations.length,
        eventHandlerCounts: Object.fromEntries(this.metrics.eventHandlers)
      }
    };
  }

  reset() {
    this.metrics.reflowCount = 0;
    this.metrics.repaintCount = 0;
    this.metrics.mutations = [];
  }
}

DOMPerformanceMonitor.instance = new DOMPerformanceMonitor();
```

---

## 与同类技术对比

### DOM 操作库对比

| 特性 | 原生 DOM | jQuery | Zepto | Cash |
|------|---------|--------|-------|------|
| 包大小 | 0 KB | 87 KB | 28 KB | 6 KB |
| 选择器 | querySelector | Sizzle | Sizzle | querySelector |
| 链式调用 | ❌ | ✅ | ✅ | ✅ |
| 动画支持 | ❌ | ✅ | ✅ | 部分 |
| 事件委托 | 手动 | ✅ | ✅ | ✅ |
| DOM 遍历 | ✅ | ✅ | ✅ | ✅ |
| 现代 API | ✅ | ❌ | ❌ | ✅ |
| 移动端优化 | - | 一般 | 优秀 | 优秀 |
| 学习曲线 | 陡峭 | 平缓 | 平缓 | 平缓 |

### BOM 功能对比

| 功能 | 标准支持 | Chrome | Firefox | Safari | Edge |
|------|---------|--------|---------|--------|------|
| window.resize | ✅ | ✅ | ✅ | ✅ | ✅ |
| window.location | ✅ | ✅ | ✅ | ✅ | ✅ |
| window.history | ✅ | ✅ | ✅ | ✅ | ✅ |
| window.navigator | ✅ | ✅ | ✅ | ✅ | ✅ |
| localStorage | ✅ | ✅ | ✅ | ✅ | ✅ |
| sessionStorage | ✅ | ✅ | ✅ | ✅ | ✅ |
| IndexedDB | ✅ | ✅ | ✅ | ✅ | ✅ |
| Service Worker | ✅ | ✅ | ✅ | ✅ | ✅ |
| WebSocket | ✅ | ✅ | ✅ | ✅ | ✅ |
| WebRTC | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## 常见问题与解决方案

### 问题 1：DOM 操作导致性能下降

**症状**：页面卡顿，动画不流畅

**原因**：频繁的 DOM 操作触发大量的重排和重绘

**解决方案**：

```javascript
// 问题代码
function updateList(items) {
  const list = document.getElementById('list');
  list.innerHTML = ''; // 触发重排
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item.name;
    list.appendChild(li); // 每添加一个就触发重排
  });
}

// 优化方案 1：使用 DocumentFragment
function updateListOptimized1(items) {
  const list = document.getElementById('list');
  const fragment = document.createDocumentFragment();

  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item.name;
    fragment.appendChild(li);
  });

  list.innerHTML = '';
  list.appendChild(fragment); // 只触发一次重排
}

// 优化方案 2：先隐藏，后显示
function updateListOptimized2(items) {
  const list = document.getElementById('list');
  list.style.display = 'none'; // 隐藏，避免重排

  list.innerHTML = '';
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item.name;
    list.appendChild(li);
  });

  list.style.display = ''; // 显示，只触发一次重排
}

// 优化方案 3：使用 CSS 类切换
function updateListOptimized3(items) {
  const list = document.getElementById('list');
  list.classList.add('loading'); // CSS 中设置 display: none

  list.innerHTML = '';
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item.name;
    list.appendChild(li);
  });

  requestAnimationFrame(() => {
    list.classList.remove('loading'); // 在下一帧显示
  });
}
```

### 问题 2：事件监听器内存泄漏

**症状**：长时间使用后内存持续增长

**原因**：未正确移除事件监听器，或引用了已删除的元素

**解决方案**：

```javascript
// 问题代码
class MemoryLeakExample {
  constructor() {
    this.data = [];
    this.init();
  }

  init() {
    const button = document.getElementById('btn');

    // 问题：使用箭头函数，无法移除
    button.addEventListener('click', () => {
      this.handleClick();
    });
  }

  handleClick() {
    // 访问 this.data，导致闭包引用
  }

  // 无法正确清理
  destroy() {
    // 试图移除，但无法匹配原来的监听器
    document.getElementById('btn').removeEventListener('click', ???);
  }
}

// 优化方案
class OptimizedExample {
  constructor() {
    this.data = [];
    this.handleClick = this.handleClick.bind(this);
    this.init();
  }

  init() {
    const button = document.getElementById('btn');
    button.addEventListener('click', this.handleClick);
  }

  handleClick() {
    console.log('Clicked');
  }

  destroy() {
    const button = document.getElementById('btn');
    if (button) {
      button.removeEventListener('click', this.handleClick);
    }
    this.data = null;
  }
}

// 最佳实践：使用 AbortController
class BestPracticeExample {
  constructor() {
    this.controller = new AbortController();
    this.init();
  }

  init() {
    const button = document.getElementById('btn');
    button.addEventListener('click', () => {
      console.log('Clicked');
    }, { signal: this.controller.signal });
  }

  destroy() {
    this.controller.abort(); // 一行代码清理所有监听器
  }
}
```

### 问题 3：跨域 iframe 通信

**症状**：无法从父页面访问 iframe 内的 DOM

**解决方案**：

```javascript
// iframe 子页面
window.addEventListener('message', event => {
  if (event.origin !== 'https://parent.com') return;

  if (event.data.type === 'GET_DATA') {
    const data = {
      title: document.title,
      user: window.currentUser
    };
    event.source.postMessage({ type: 'DATA_RESPONSE', data }, event.origin);
  }
});

// 父页面
const iframe = document.getElementById('child-frame');

function getIframeData() {
  return new Promise((resolve, reject) => {
    const timeout = setTimeout(() => {
      reject(new Error('Timeout'));
    }, 5000);

    const handler = event => {
      if (event.origin !== 'https://child.com') return;
      if (event.data.type === 'DATA_RESPONSE') {
        clearTimeout(timeout);
        window.removeEventListener('message', handler);
        resolve(event.data.data);
      }
    };

    window.addEventListener('message', handler);
    iframe.contentWindow.postMessage(
      { type: 'GET_DATA' },
      'https://child.com'
    );
  });
}
```

### 问题 4：动态加载内容的样式不生效

**症状**：动态添加的 DOM 元素没有样式

**原因**：CSS 选择器优先级问题，或样式表未加载完成

**解决方案**：

```javascript
// 检查样式是否已加载
function waitForStyles() {
  return new Promise(resolve => {
    if (document.styleSheets.length > 0) {
      resolve();
      return;
    }

    const observer = new MutationObserver(mutations => {
      if (document.styleSheets.length > 0) {
        observer.disconnect();
        resolve();
      }
    });

    observer.observe(document.head, { childList: true });
  });
}

// 确保元素在样式应用后才显示
async function createStyledElement() {
  await waitForStyles();

  const element = document.createElement('div');
  element.className = 'my-styled-class';
  element.textContent = 'Content';

  // 使用 requestAnimationFrame 确保在下一帧渲染
  requestAnimationFrame(() => {
    document.body.appendChild(element);
  });
}

// 使用 CSS 变量而非计算样式
element.style.setProperty('--dynamic-color', 'red');
```

---

## 实战项目示例

### 项目 1：可编辑数据表格

**需求**：创建一个功能完整的数据表格，支持排序、筛选、分页和行编辑

```javascript
class EditableDataTable {
  constructor(container, options = {}) {
    this.container = container;
    this.data = options.data || [];
    this.columns = options.columns || [];
    this.pageSize = options.pageSize || 10;
    this.currentPage = 1;
    this.sortColumn = null;
    this.sortDirection = 'asc';
    this.filters = {};

    this.init();
  }

  init() {
    this.render();
    this.attachEvents();
  }

  render() {
    const start = (this.currentPage - 1) * this.pageSize;
    const end = start + this.pageSize;
    let displayData = this.getFilteredData();
    displayData = this.getSortedData(displayData);
    this.paginatedData = displayData.slice(start, end);
    this.totalPages = Math.ceil(displayData.length / this.pageSize);

    this.container.innerHTML = `
      <div class="data-table-wrapper">
        ${this.renderFilters()}
        ${this.renderTable()}
        ${this.renderPagination()}
      </div>
    `;

    this.attachRowEvents();
  }

  renderFilters() {
    return `
      <div class="filters">
        ${this.columns.map(col => `
          <input
            type="text"
            placeholder="Filter ${col.header}"
            data-column="${col.key}"
            class="filter-input"
            value="${this.filters[col.key] || ''}"
          />
        `).join('')}
        <button class="clear-filters">Clear Filters</button>
      </div>
    `;
  }

  renderTable() {
    return `
      <table class="data-table">
        <thead>
          <tr>
            ${this.columns.map(col => `
              <th data-column="${col.key}">
                ${col.header}
                ${this.sortColumn === col.key
                  ? (this.sortDirection === 'asc' ? ' ▲' : ' ▼')
                  : ''}
              </th>
            `).join('')}
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          ${this.paginatedData.map((row, index) => `
            <tr data-index="${(this.currentPage - 1) * this.pageSize + index}">
              ${this.columns.map(col => `
                <td data-column="${col.key}">
                  ${this.renderCell(row, col, index)}
                </td>
              `).join('')}
              <td>
                <button class="edit-row" data-index="${index}">Edit</button>
                <button class="delete-row" data-index="${index}">Delete</button>
              </td>
            </tr>
          `).join('')}
        </tbody>
      </table>
    `;
  }

  renderCell(row, col, index) {
    if (col.editable) {
      return `<span class="cell-value">${row[col.key]}</span>`;
    }
    return row[col.key];
  }

  renderPagination() {
    return `
      <div class="pagination">
        <button
          class="prev-page"
          ${this.currentPage === 1 ? 'disabled' : ''}
        >
          Previous
        </button>
        <span class="page-info">
          Page ${this.currentPage} of ${this.totalPages}
        </span>
        <button
          class="next-page"
          ${this.currentPage === this.totalPages ? 'disabled' : ''}
        >
          Next
        </button>
      </div>
    `;
  }

  attachEvents() {
    // 排序
    this.container.querySelectorAll('th[data-column]').forEach(th => {
      th.addEventListener('click', () => this.handleSort(th.dataset.column));
    });

    // 筛选
    this.container.querySelectorAll('.filter-input').forEach(input => {
      input.addEventListener('input', debounce(e => {
        this.filters[e.target.dataset.column] = e.target.value;
        this.currentPage = 1;
        this.render();
      }, 300));
    });

    // 清空筛选
    this.container.querySelector('.clear-filters')?.addEventListener('click', () => {
      this.filters = {};
      this.currentPage = 1;
      this.render();
    });

    // 分页
    this.container.querySelector('.prev-page')?.addEventListener('click', () => {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.render();
      }
    });

    this.container.querySelector('.next-page')?.addEventListener('click', () => {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.render();
      }
    });
  }

  attachRowEvents() {
    // 编辑
    this.container.querySelectorAll('.edit-row').forEach(btn => {
      btn.addEventListener('click', e => this.handleEdit(parseInt(e.target.dataset.index)));
    });

    // 删除
    this.container.querySelectorAll('.delete-row').forEach(btn => {
      btn.addEventListener('click', e => this.handleDelete(parseInt(e.target.dataset.index)));
    });
  }

  handleSort(column) {
    if (this.sortColumn === column) {
      this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      this.sortColumn = column;
      this.sortDirection = 'asc';
    }
    this.render();
  }

  handleEdit(index) {
    const row = this.paginatedData[index];
    const cells = this.container.querySelectorAll(`tr[data-index="${index}"] td`);

    cells.forEach((cell, i) => {
      const col = this.columns[i];
      if (col?.editable) {
        const currentValue = row[col.key];
        cell.innerHTML = `
          <input type="text" value="${currentValue}" class="edit-input" />
        `;
      }
    });

    const saveHandler = () => {
      cells.forEach((cell, i) => {
        const col = this.columns[i];
        if (col?.editable) {
          const input = cell.querySelector('input');
          if (input) {
            row[col.key] = input.value;
            cell.textContent = input.value;
          }
        }
      });
      this.container.removeEventListener('click', saveHandler);
    };

    setTimeout(() => {
      this.container.addEventListener('click', saveHandler);
    }, 0);
  }

  handleDelete(index) {
    const actualIndex = (this.currentPage - 1) * this.pageSize + index;
    this.data.splice(actualIndex, 1);
    this.render();
  }

  getFilteredData() {
    return this.data.filter(row => {
      return Object.entries(this.filters).every(([column, filterValue]) => {
        if (!filterValue) return true;
        const cellValue = String(row[column]).toLowerCase();
        return cellValue.includes(filterValue.toLowerCase());
      });
    });
  }

  getSortedData(data) {
    if (!this.sortColumn) return data;

    return [...data].sort((a, b) => {
      const aVal = a[this.sortColumn];
      const bVal = b[this.sortColumn];

      if (aVal < bVal) return this.sortDirection === 'asc' ? -1 : 1;
      if (aVal > bVal) return this.sortDirection === 'asc' ? 1 : -1;
      return 0;
    });
  }
}

// 使用示例
const table = new EditableDataTable(
  document.getElementById('table-container'),
  {
    data: [
      { id: 1, name: 'John', email: 'john@example.com', status: 'active' },
      { id: 2, name: 'Jane', email: 'jane@example.com', status: 'inactive' },
      // ... 更多数据
    ],
    columns: [
      { key: 'id', header: 'ID', editable: false },
      { key: 'name', header: 'Name', editable: true },
      { key: 'email', header: 'Email', editable: true },
      { key: 'status', header: 'Status', editable: true }
    ],
    pageSize: 10
  }
);
```

### 项目 2：拖拽排序列表

```javascript
class DragSortableList {
  constructor(container, options = {}) {
    this.container = container;
    this.items = [...container.children];
    this.draggedItem = null;
    this.draggedIndex = null;
    this.onSort = options.onSort || (() => {});
    this.animationClass = options.animationClass || 'dragging';

    this.init();
  }

  init() {
    this.items.forEach((item, index) => {
      item.draggable = true;
      item.dataset.index = index;

      item.addEventListener('dragstart', this.handleDragStart.bind(this));
      item.addEventListener('dragenter', this.handleDragEnter.bind(this));
      item.addEventListener('dragover', this.handleDragOver.bind(this));
      item.addEventListener('dragleave', this.handleDragLeave.bind(this));
      item.addEventListener('drop', this.handleDrop.bind(this));
      item.addEventListener('dragend', this.handleDragEnd.bind(this));
    });
  }

  handleDragStart(e) {
    this.draggedItem = e.target;
    this.draggedIndex = parseInt(e.target.dataset.index);

    e.target.classList.add(this.animationClass);
    e.dataTransfer.effectAllowed = 'move';
    e.dataTransfer.setData('text/plain', this.draggedIndex);

    // 添加拖拽视觉反馈
    setTimeout(() => {
      e.target.style.opacity = '0.5';
    }, 0);
  }

  handleDragOver(e) {
    e.preventDefault();
    e.dataTransfer.dropEffect = 'move';

    if (e.target === this.draggedItem) return;

    const rect = e.target.getBoundingClientRect();
    const midY = rect.top + rect.height / 2;

    // 移除之前的指示器
    this.container.querySelectorAll('.drop-indicator').forEach(el => el.remove());

    // 添加新的指示器
    const indicator = document.createElement('div');
    indicator.className = 'drop-indicator';

    if (e.clientY < midY) {
      indicator.style.height = '0';
      e.target.parentElement.insertBefore(indicator, e.target);
    } else {
      indicator.style.height = '0';
      e.target.parentElement.insertBefore(indicator, e.target.nextSibling);
    }
  }

  handleDragEnter(e) {
    e.preventDefault();
  }

  handleDragLeave(e) {
    // 如果离开容器，移除指示器
    if (!e.relatedTarget || !this.container.contains(e.relatedTarget)) {
      this.container.querySelectorAll('.drop-indicator').forEach(el => el.remove());
    }
  }

  handleDrop(e) {
    e.preventDefault();

    if (e.target === this.draggedItem) return;

    const rect = e.target.getBoundingClientRect();
    const midY = rect.top + rect.height / 2;
    const dropIndex = parseInt(e.target.dataset.index);

    // 计算实际插入位置
    let insertIndex = e.clientY < midY ? dropIndex : dropIndex + 1;

    // 调整插入索引
    if (this.draggedIndex < insertIndex) {
      insertIndex--;
    }

    // 执行移动
    if (insertIndex !== this.draggedIndex) {
      const movedItems = [...this.container.children];
      const [removed] = movedItems.splice(this.draggedIndex, 1);
      movedItems.splice(insertIndex, 0, removed);

      // 清空容器
      this.container.innerHTML = '';

      // 重新添加元素
      movedItems.forEach((item, index) => {
        item.dataset.index = index;
        this.container.appendChild(item);
      });

      this.onSort({
        oldIndex: this.draggedIndex,
        newIndex: insertIndex,
        items: movedItems.map(item => item.dataset.id)
      });
    }
  }

  handleDragEnd(e) {
    e.target.classList.remove(this.animationClass);
    e.target.style.opacity = '1';

    // 移除所有指示器
    this.container.querySelectorAll('.drop-indicator').forEach(el => el.remove());

    // 更新索引
    [...this.container.children].forEach((item, index) => {
      item.dataset.index = index;
    });
  }
}

// 使用示例
const list = new DragSortableList(
  document.getElementById('sortable-list'),
  {
    onSort: ({ oldIndex, newIndex, items }) => {
      console.log(`Moved from ${oldIndex} to ${newIndex}`);
      console.log('New order:', items);
    }
  }
);
```

---

## 与同类技术对比

### DOM 操作库对比

原生 DOM API 与各种封装库的对比分析：

#### 原生 API vs jQuery

| 维度 | 原生 DOM API | jQuery |
|------|-------------|--------|
| **文件大小** | 0 KB（浏览器内置） | 87 KB（压缩后） |
| **学习曲线** | 较陡峭 | 平缓 |
| **选择器** | `querySelector`/`querySelectorAll` | Sizzle 引擎 |
| **DOM 操作** | 原生方法 | 跨浏览器兼容方法 |
| **动画** | `requestAnimationFrame` | `$.animate()` |
| **事件处理** | `addEventListener` | `$.on()` |
| **链式调用** | 不支持 | 支持 |
| **现代特性** | 完整支持 | 部分支持 |
| **维护状态** | 活跃（浏览器标准） | 停止维护（仅安全更新） |

```javascript
// 原生 DOM API
const buttons = document.querySelectorAll('.btn');
buttons.forEach(btn => {
  btn.addEventListener('click', handleClick);
});

// jQuery
$('.btn').on('click', handleClick);

// 原生 DOM 操作
const div = document.createElement('div');
div.className = 'card';
div.textContent = 'Hello';
div.style.color = 'red';
parent.appendChild(div);

// jQuery
$('<div>', { class: 'card', text: 'Hello' }).css('color', 'red').appendTo(parent);
```

#### 原生 API vs React/Vue

| 维度 | 原生 DOM | React | Vue |
|------|----------|-------|-----|
| **更新机制** | 直接操作 | 虚拟 DOM | 响应式 |
| **状态管理** | 原生变量 | setState/useState | ref/reactive |
| **组件化** | 无内置 | 组件系统 | 组件系统 |
| **性能** | 手动优化 | 自动优化 | 自动优化 |
| **适用场景** | 简单页面、小型项目 | 复杂应用 | 复杂应用 |
| **Bundle 体积** | 0 KB | 40 KB | 35 KB |

#### DOM 操作性能对比

| 操作类型 | 原生 API | jQuery | 框架 |
|---------|----------|--------|------|
| 单元素查询 | `querySelector` (快) | `$()` (中) | 内部优化 |
| 批量查询 | `querySelectorAll` (快) | `$()` (中) | 内部优化 |
| 属性修改 | 直接设置 (快) | `.attr()` (中) | 虚拟 DOM (需diff) |
| 元素创建 | `createElement` (快) | `$()` (中) | JSX (需编译) |
| 批量插入 | `DocumentFragment` (快) | `.append()` (中) | 批量更新 (快) |
| 事件绑定 | `addEventListener` (快) | `.on()` (中) | 委托 (快) |

### BOM 功能对比

#### 浏览器兼容性一览

| API | Chrome | Firefox | Safari | Edge | IE 11 |
|-----|--------|---------|--------|-------|--------|
| `window.innerWidth` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `navigator.hardwareConcurrency` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `navigator.deviceMemory` | ✅ | ❌ | ❌ | ✅ | ❌ |
| `performance.now()` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `crypto.getRandomValues()` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `localStorage` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `IndexedDB` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `WebSocket` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `Service Worker` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `IntersectionObserver` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `ResizeObserver` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `MutationObserver` | ✅ | ✅ | ✅ | ✅ | ✅ |

### 观察者 API 对比

#### IntersectionObserver vs 传统滚动检测

| 维度 | IntersectionObserver | 传统 scroll 事件 |
|------|---------------------|-------------------|
| **性能** | 优秀（浏览器优化） | 差（持续触发） |
| **精度** | 像素级 | 依赖节流 |
| **电池消耗** | 低 | 高 |
| **API 复杂度** | 简单 | 需自行计算 |
| **兼容性** | 现代浏览器 | 所有浏览器 |
| **移动端支持** | ✅ | ✅ |

```javascript
// 传统方式（性能差）
let ticking = false;
window.addEventListener('scroll', () => {
  if (!ticking) {
    window.requestAnimationFrame(() => {
      const rect = element.getBoundingClientRect();
      const isVisible = rect.top < window.innerHeight && rect.bottom > 0;
      if (isVisible) loadContent();
      ticking = false;
    });
    ticking = true;
  }
});

// IntersectionObserver（性能优）
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadContent();
      observer.unobserve(entry.target);
    }
  });
}, { threshold: 0.1 });

observer.observe(element);
```

#### ResizeObserver vs window.resize

| 维度 | ResizeObserver | window.resize |
|------|---------------|---------------|
| **触发频率** | 元素级别 | 窗口级别 |
| **性能** | 优秀 | 较差 |
| **元素监听** | 任意元素 | 仅窗口 |
| **包含 padding** | 可配置 | 否 |
| **兼容性** | 现代浏览器 | 所有浏览器 |

---

## 高级配置与技巧

### DOM 性能监控配置

#### 完整性能监控实现

```javascript
class DOMPerformanceMonitor {
  constructor() {
    this.metrics = {
      reflowCount: 0,
      repaintCount: 0,
      mutations: [],
      eventHandlers: new Map(),
      largestReflow: { size: 0, timestamp: 0, stack: '' }
    };

    this.setupMonitoring();
  }

  setupMonitoring() {
    // 监控重排
    const originalGetBCR = Element.prototype.getBoundingClientRect;
    Element.prototype.getBoundingClientRect = function(...args) {
      const result = originalGetBCR.apply(this, args);
      this._monitorReflow();
      return result;
    };

    // 监控样式计算
    const originalCS = window.getComputedStyle;
    window.getComputedStyle = function(element, pseudoElt) {
      this._monitorStyleCalculation();
      return originalCS.apply(this, arguments);
    };

    // 监控 MutationObserver
    this.setupMutationObserver();
    
    // 监控事件处理
    this.setupEventMonitoring();
  }

  setupMutationObserver() {
    const observer = new MutationObserver(mutations => {
      mutations.forEach(mutation => {
        this.metrics.mutations.push({
          type: mutation.type,
          target: mutation.target.nodeName,
          addedNodes: mutation.addedNodes.length,
          removedNodes: mutation.removedNodes.length,
          attributeName: mutation.attributeName,
          timestamp: performance.now()
        });
        
        // 检测大量 DOM 变更
        if (mutation.addedNodes.length > 10 || mutation.removedNodes.length > 10) {
          console.warn('Large DOM mutation detected:', mutation);
        }
      });
    });

    observer.observe(document.body, {
      childList: true,
      subtree: true,
      attributes: true,
      attributeOldValue: true,
      characterData: true,
      characterDataOldValue: true
    });
  }

  setupEventMonitoring() {
    const originalAddEventListener = EventTarget.prototype.addEventListener;
    EventTarget.prototype.addEventListener = function(type, listener, options) {
      const key = `${this.nodeName || this.constructor.name}-${type}`;
      const count = this.metrics?.eventHandlers?.get(key) || 0;
      
      // 检测重复事件监听器
      if (count > 5) {
        console.warn(`Element ${key} has ${count} event listeners for ${type}`);
      }
      
      return originalAddEventListener.call(this, type, listener, options);
    };
  }

  _monitorReflow() {
    this.metrics.reflowCount++;
    
    // 记录最大重排
    const now = performance.now();
    if (now - this.metrics.largestReflow.timestamp > 1000) {
      const error = new Error();
      this.metrics.largestReflow = {
        size: this.metrics.reflowCount,
        timestamp: now,
        stack: error.stack
      };
    }
  }

  _monitorStyleCalculation() {
    this.metrics.repaintCount++;
  }

  getReport() {
    return {
      summary: {
        totalReflows: this.metrics.reflowCount,
        totalRepaints: this.metrics.repaintCount,
        totalMutations: this.metrics.mutations.length,
        avgMutationsPerSecond: this.metrics.mutations.length / 
          ((performance.now() - (this.metrics.startTime || performance.now())) / 1000)
      },
      largestReflow: this.metrics.largestReflow,
      recommendations: this.generateRecommendations()
    };
  }

  generateRecommendations() {
    const recommendations = [];
    
    if (this.metrics.reflowCount > 100) {
      recommendations.push({
        severity: 'high',
        message: '检测到大量重排操作，建议批量 DOM 更新'
      });
    }
    
    if (this.metrics.mutations.length > 1000) {
      recommendations.push({
        severity: 'medium',
        message: 'DOM 变更过于频繁，考虑使用防抖或节流'
      });
    }
    
    return recommendations;
  }

  reset() {
    this.metrics.reflowCount = 0;
    this.metrics.repaintCount = 0;
    this.metrics.mutations = [];
    this.metrics.startTime = performance.now();
  }
}

// 使用示例
const monitor = new DOMPerformanceMonitor();

// 运行测试
document.getElementById('test-button').click();

// 获取报告
console.table(monitor.getReport());
```

### Web Animations API 深度应用

#### 高级动画控制

```javascript
class AdvancedAnimator {
  constructor(element) {
    this.element = element;
    this.animations = new Map();
  }

  // 并行动画
  parallel(keyframes, options) {
    const animation = this.element.animate(keyframes, {
      ...options,
      composite: 'add' // 与现有动画叠加
    });
    this.animations.set('parallel', animation);
    return animation;
  }

  // 序列动画
  sequence(animations) {
    const group = [];
    let prevEndTime = 0;

    animations.forEach((anim, index) => {
      const animation = this.element.animate(anim.keyframes, {
        ...anim.options,
        delay: prevEndTime + (anim.options?.delay || 0)
      });
      group.push(animation);
      prevEndTime = animation.startTime + 
        (anim.options?.duration || 300) +
        (anim.options?.delay || 0);
    });

    return group;
  }

  // 可中断动画
  interruptible(keyframes, options) {
    // 取消现有动画
    if (this.currentAnimation) {
      this.currentAnimation.cancel();
    }

    this.currentAnimation = this.element.animate(keyframes, {
      ...options,
      composite: 'replace'
    });

    return this.currentAnimation;
  }

  // 弹性动画
  spring(keyframes, springConfig = {}) {
    const {
      stiffness = 100,
      damping = 10,
      mass = 1
    } = springConfig;

    // 简化的弹性动画实现
    const duration = 1000;
    const startTime = performance.now();
    
    const animation = new Animation(
      new KeyframeEffect(
        this.element,
        keyframes.map((kf, i) => ({
          ...kf,
          easing: 'linear'
        })),
        { duration, fill: 'forwards' }
      ),
      document.timeline
    );

    // 应用弹性缓动
    const proxy = new Proxy(animation, {
      get(target, prop) {
        if (prop === 'currentTime') {
          const t = target.currentTime;
          const progress = t / duration;
          // 简化的弹性计算
          const dampedProgress = 1 - Math.exp(-stiffness * progress) * 
            Math.cos(damping * progress);
          return dampedProgress * duration;
        }
        return target[prop];
      }
    });

    proxy.play();
    return proxy;
  }
}

// 使用示例
const element = document.querySelector('.animated');
const animator = new AdvancedAnimator(element);

// 并行动画
animator.parallel([
  { transform: 'translateX(100px)' },
  { opacity: 0 }
], { duration: 500, easing: 'ease-out' });

// 序列动画
animator.sequence([
  { keyframes: [{ transform: 'scale(1)' }, { transform: 'scale(1.2)' }], 
    options: { duration: 200 } },
  { keyframes: [{ transform: 'scale(1.2)' }, { transform: 'scale(1)' }], 
    options: { duration: 200 } }
]);
```

### Web Components 深度应用

#### 自定义元素实现

```javascript
// 定义自定义元素
class MyCounter extends HTMLElement {
  static get observedAttributes() {
    return ['count', 'step'];
  }

  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this._count = 0;
    this._step = 1;
  }

  connectedCallback() {
    this.render();
    this.attachEvents();
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue !== newValue) {
      if (name === 'count') this._count = parseInt(newValue, 10);
      if (name === 'step') this._step = parseInt(newValue, 10);
      this.updateDisplay();
    }
  }

  render() {
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: inline-flex;
          align-items: center;
          gap: 8px;
          padding: 8px 16px;
          background: var(--bg-color, #f0f0f0);
          border-radius: 8px;
          font-family: system-ui;
        }
        button {
          padding: 4px 12px;
          cursor: pointer;
        }
        .count {
          min-width: 40px;
          text-align: center;
          font-size: 18px;
          font-weight: bold;
        }
      </style>
      <button class="decrement">-</button>
      <span class="count">${this._count}</span>
      <button class="increment">+</button>
    `;
  }

  attachEvents() {
    this.shadowRoot.querySelector('.increment')
      .addEventListener('click', () => this.increment());
    this.shadowRoot.querySelector('.decrement')
      .addEventListener('click', () => this.decrement());
  }

  increment() {
    this._count += this._step;
    this.setAttribute('count', this._count);
    this.dispatchEvent(new CustomEvent('change', { 
      detail: { count: this._count }
    }));
  }

  decrement() {
    this._count -= this._step;
    this.setAttribute('count', this._count);
    this.dispatchEvent(new CustomEvent('change', { 
      detail: { count: this._count }
    }));
  }

  updateDisplay() {
    const countEl = this.shadowRoot.querySelector('.count');
    if (countEl) countEl.textContent = this._count;
  }
}

// 注册自定义元素
customElements.define('my-counter', MyCounter);

// 使用
// <my-counter count="0" step="5"></my-counter>
```

#### Shadow DOM 样式隔离

```javascript
class StyledComponent extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
  }

  connectedCallback() {
    // 方式 1：内联样式
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: block;
          padding: 16px;
          background: white;
          border-radius: 8px;
          box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
        :host([dark]) {
          background: #1a1a1a;
          color: white;
        }
        :host([dark]) .content {
          border-color: #333;
        }
        .content {
          border: 1px solid #eee;
          padding: 12px;
        }
      </style>
      <div class="content">
        <slot></slot>
      </div>
    `;
  }
}

// 方式 2：外部样式表（需在构造器中创建 link）
constructor() {
  super();
  this.attachShadow({ mode: 'open' });
  
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = '/components/my-component.css';
  
  this.shadowRoot.appendChild(link);
}
```

---

## 总结

DOM 和 BOM 提供了丰富的 API 来操作网页和控制浏览器行为。DOM 树形结构的理解是高效操作页面的基础，而现代观察者 API（IntersectionObserver、ResizeObserver、MutationObserver）则让我们能够以声明式的方式响应各种变化，避免了传统轮询的性能开销。掌握这些 API，能够让你的前端代码更加高效、可维护，同时提供更好的用户体验。

核心要点回顾：

**DOM 操作最佳实践**包括使用 `DocumentFragment` 减少重排、使用事件委托减少监听器数量、使用 `requestAnimationFrame` 同步动画帧、以及使用现代选择器 API 替代旧的 `getElementsBy*` 方法。这些技术可以显著提升页面性能，特别是在处理大量 DOM 元素时。

**BOM 关键 API**涵盖了 `window` 对象的各种属性和方法、`navigator` 的设备信息获取、`location` 的 URL 操作、以及 `history` 的 SPA 路由实现。理解这些 API 的工作原理，可以帮助开发者更好地控制浏览器行为和实现复杂的交互逻辑。

**观察者 API** 是现代 Web 开发的重要工具。`IntersectionObserver` 用于检测元素可见性，是实现懒加载和无限滚动的利器。`ResizeObserver` 用于监听元素尺寸变化，比传统的 `resize` 事件更加精确和高效。`MutationObserver` 用于监听 DOM 变化，是实现响应式数据和自动保存等功能的基础。

**性能优化**是 DOM 和 BOM 操作中永恒的话题。减少重排重绘、避免内存泄漏、优化事件处理、使用虚拟滚动等技术，都是构建高性能 Web 应用的关键。通过合理使用这些技术，可以显著提升用户体验。

> [!RELATED]
> - [[JavaScript 事件处理]] - 事件系统的深入探讨
> - [[前端性能优化]] - 性能优化的综合指南
> - [[响应式设计]] - 媒体查询和布局技巧
