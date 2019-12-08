
# React Fiber

### 為何要改架構？
1. 服務於一個最終目的：使用者體驗

3. 瀏覽器是 Single-Thread
- 瀏覽器渲染流程
![](https://miro.medium.com/max/2186/0*_qpgAjv7U3Q3X6k1.jpg)[https://developers.google.com/web/fundamentals/performance/rendering/](https://developers.google.com/web/fundamentals/performance/rendering/)

#### 原先方式 ( Stack Reconciliation )
1. 
#### 未來 ( Suspense, Concurrent Mode )
1. JIT優化

...

## Fiber 實現方式
...
## 未整理內容

#### 深度優先遍歷
...

## 遞迴 > 遍歷(單鏈表)
原先是利用遞迴來進行 diff 運算，而每進入一個函數會建立一個上下文環境，這將會有多餘的資源消耗，而改成單鏈表則能單鏈表

#### React Hook
React Hook 中紀錄 hook 執行順序的 `updateQueue` 及紀錄對應狀態的 `memonizedState` 存在於 `fiber` 中

#### React Element => React Fiber
JSX
```javascript
<Card className="card">
  <div className="card-header"></div>
</Card>
```
Compiled
```javascript
React.createElement(Card, {
  className: "card"
}, React.createElement("div", {
  className: "card-header"
}));
```
[Source](https://github.com/facebook/react/blob/master/packages/react/src/ReactElement.js#L316)

```javascript
export function createElement(type, config, children) {
  let propName;

  // props (排除內建 Props)
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  // 如果有額外的屬性
  if (config != null) {
    // 如果有 ref，檢測是否合法
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    // 如果有 key，檢測是否合法
    if (hasValidKey(config)) {
	  // 轉成字串
      key = '' + config.key;
    }
	
	// for JSX
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    
    // 只寫入非把除了內建 props 以外注入
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }
  
  // -2 是因前兩個屬性 type, config
  // 這之後的參數皆為
  const childrenLength = arguments.length - 2;
  // 如果只有一個
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    // 建立子組件陣列
    const childArray = Array(childrenLength);
    // 寫入
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
	// 開發模式
    if (__DEV__) {
      if (Object.freeze) {
        // 凍結陣列，使其不能被修改
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }

  // 如果 type 擁有 defaultProps (Class Component, Function Component)
  if (type && type.defaultProps) {
   e  = type.defaultProps;
    // 寫入預設 props
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  // 開發模式
  if (__DEV__) {
    if (key || ref) {
      const displayName =
        typeof type === 'function'
          ? type.displayName || type.name || 'Unknown'
          : type;
      if (key) {
        defineKeyPropWarningGetter(props, displayName);
      }
      if (ref) {
        defineRefPropWarningGetter(props, displayName);
      }
    }
  }
  // 建立 ReactElement
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```
簡單說明一下 `ReactCurrentOwner.current`

```javascript
import type {Fiber} from 'react-reconciler/src/ReactFiber';

const ReactCurrentOwner = {
  current: (null: null | Fiber),
};

export default ReactCurrentOwner;
```
統一都從這裡拿到物件，根據 `module` 特性，共享同一個實例

回來接續上方 Code
```javascript
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // React 內建型別系統，REACT_ELEMENT_TYPE 代表 ReactElement
    $$typeof: REACT_ELEMENT_TYPE,

	// 內建屬性
    type: type,
    key: key,
    ref: ref,
    props: props,

    // 在某些階段會被寫入值，代表目前正在建構的組件的所屬的組件(父組件)
    _owner: owner,
  };

  // TODO: 這段暫時跳過
  if (__DEV__) {
    // The validation flag is currently mutative. We put it on
    // an external backing store so that we can freeze the whole object.
    // This can be replaced with a WeakMap once they are implemented in
    // commonly used development environments.
    element._store = {};

    // To make comparing ReactElements easier for testing purposes, we make
    // the validation flag non-enumerable (where possible, which should
    // include every environment we run tests in), so the test framework
    // ignores it.
    Object.defineProperty(element._store, 'validated', {
      configurable: false,
      enumerable: false,
      writable: true,
      value: false,
    });
    // self and source are DEV only properties.
    Object.defineProperty(element, '_self', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: self,
    });
    // Two elements created in two different places should be considered
    // equal for testing purposes and therefore we hide it from enumeration.
    Object.defineProperty(element, '_source', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: source,
    });
    if (Object.freeze) {
      Object.freeze(element.props);
      Object.freeze(element);
    }
  }

  return element;
};
```
建立 `ReactElement` 的流程簡單先到這裡，接下來我們跳到 `react-dom` 來看整個渲染流程。

這是一個常見的入口

```javascript
ReactDOM.render(
  <App/>,
  document.getElementById("root")
);
```


`react-dom/index.js`
```javascript
const ReactDOM = require('./src/client/ReactDOM');
module.exports = ReactDOM.default || ReactDOM;
```

`react-dom/client/ReactDom.js`
```javascript
import {
  findDOMNode,
  render,
  hydrate,
  unstable_renderSubtreeIntoContainer,
  unmountComponentAtNode,
} from './ReactDOMLegacy';

const ReactDOM: Object = {
  createPortal,

  // Legacy
  findDOMNode,
  hydrate,
  render,
  unstable_renderSubtreeIntoContainer,
  unmountComponentAtNode,
```
擷取其中一小段，繼續跟進

`react-dom/client/ReactDOMLegacy.js`
```
export function render(
  element: React$Element<any>,
  container: DOMContainer,
  callback: ?Function,
) {
  invariant(
    isValidContainer(container),
    'Target container is not a DOM element.',
  );
  if (__DEV__) {
  RootontainerernRoot null  const isModernRoot =
      isContainerMarkedAs// 略
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}
```

```javascript
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  container: DOMContainer,
  forceHydrate: boolean,
  callback: ?Function,
) {
  if (__DEV__) {
    topLevelUpdateWarnings(container);
    warnOnInvalidCallback(callback === undefined ? null : callback, 'render');
  }

  // TODO: Without `any` type, Flow says "Property cannot be accessed on any
  // member of intersection type." Whyyyyyy.
  let root: RootType = (container._reactRootContainer: any);
  let fiberRoot;
  if (!root) {
    // Initial mount
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // Initial mount should not be batched.
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // Update
    updateContainer(children, fiberRoot, parentComponent, callback);
  }
  return getPublicRootInstance(fiberRoot);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NjQ3NDk4NjUsLTEyMDAwMjg2NjIsMT
Q4MDA3MDU3MCwxNDI3ODM3MjQxLDEwMjA5NjE0MjcsMTE4ODQ5
NjMwMywtMTU1NTg2MjI0OSw5NzY1MDgzMzgsLTEzMjI4NjEwMC
w1ODk1NTY3NjgsLTE4NTgxNDAwMzgsLTExMDI5OTQwMzYsLTgx
OTAwNzg0NCwxMjQ1MDc1ODI4LDEzNDc2NTQxOTAsMjA3OTkxMj
A3NCwtMTIwNDUwNjQ4NywtMTU5MTkzOTQyOV19
-->