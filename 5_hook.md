"""

//js逆向通用hook集合，主要还是（console.log），结合浏览器调试使用

//1.Cookie Hook 定位 Cookie 中关键参数生成位置

const originalCookieDesc = Object.getOwnPropertyDescriptor(Document.prototype, 'cookie');
Object.defineProperty(document, 'cookie', {
    get: function() {
        const cookies = originalCookieDesc.get.call(this);
        console.trace('读取cookie:', cookies);  // 使用trace获取调用栈
        return cookies;
    },
    set: function(val) {// 过滤特定cookie的设置
        if (val.includes('your_target_key')) {
            console.log('设置关键cookie:', val);
            debugger;  // 自动断点
        }
        return originalCookieDesc.set.call(this, val);
    },
    configurable: true
});



//2. 针对headers的hook，这里针对（Authorization）参数

// 拦截所有请求头设置操作
(function() {
    var originalSetRequestHeader = XMLHttpRequest.prototype.setRequestHeader;
    
    XMLHttpRequest.prototype.setRequestHeader = function(header, value) {
        if (header.toLowerCase() === 'authorization') {
            console.log('设置 Authorization 头:', value);
            // 可以在这里修改 value
            // value = 'Bearer modified_token';
        }
        return originalSetRequestHeader.call(this, header, value);
    };
})();




//3.浏览器调试时候，跳过debugger的hook【涉及哪个用哪个】

#通用跳过debugger

## 1. 禁用 debugger 语句

```javascript
// 重写 debugger 语句使其无效
var _debugger = window.debugger;
window.debugger = function(){};
```

## 2. Hook console.debug 方法

```javascript
// 禁用所有 console.debug 输出
console.debug = function(){};
```

## 3. 禁用开发者工具检测

许多网站会检测开发者工具是否打开：

```javascript
// 禁用常见的开发者工具检测
Object.defineProperty(window, 'devtools', { get: () => false });
Object.defineProperty(window, 'webkitStorageInfo', { get: () => false });
Object.defineProperty(window, 'ondevtoolschange', { get: () => false });
```

## 4. 重写 Date 和 performance 方法

一些反调试会使用时间差检测：

```javascript
// 保持时间一致防止检测
const _Date = Date;
Date = function() {
  return new _Date(0); // 返回固定时间
};
Date.now = () => 0;
performance.now = () => 0;
```

## 5. 禁用断点调试检测

```javascript
// 防止通过 Function.toString 检测
Function.prototype.toString = function() {
  return "function() { [native code] }";
};

// 禁用 debugger 功能
Object.defineProperty(window, 'Debugger', { get: () => {} });
```

## 6. 完整的反反调试脚本

```javascript
(function() {
  'use strict';
  
  // 1. 禁用 debugger 语句
  window.debugger = function(){};
  
  // 2. 禁用 console 调试方法
  console.debug = function(){};
  console.log = function(){};
  console.warn = function(){};
  console.error = function(){};
  
  // 3. 禁用开发者工具检测
  Object.defineProperty(window, 'devtools', { get: () => false });
  Object.defineProperty(window, 'webkitStorageInfo', { get: () => false });
  Object.defineProperty(window, 'ondevtoolschange', { get: () => false });
  
  // 4. 固定时间相关方法
  const _Date = Date;
  Date = function() { return new _Date(0); };
  Date.now = () => 0;
  performance.now = () => 0;
  
  // 5. 防止函数检测
  Function.prototype.toString = function() {
    return "function() { [native code] }";
  };
  
  // 6. 禁用其他常见检测方式
  Object.defineProperty(document, 'hidden', { get: () => true });
  Object.defineProperty(document, 'visibilityState', { get: () => 'visible' });
  
  console.log('所有调试检测已被禁用');
})();
```

## 7. 使用 Chrome 扩展注入

创建一个 Chrome 扩展的 content script 来注入这些 hook：

```json
// manifest.json
{
  "name": "Anti Debugger Detection",
  "version": "1.0",
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["antidebug.js"],
    "run_at": "document_start"
  }]
}
```



"""
