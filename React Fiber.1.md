
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

接續上方 Code
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

    // Record the component responsible for creating this element.
    _owner: owner,
  };

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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA1Mjg5MzgyLC0xNTU1ODYyMjQ5LDk3Nj
UwODMzOCwtMTMyMjg2MTAwLDU4OTU1Njc2OCwtMTg1ODE0MDAz
OCwtMTEwMjk5NDAzNiwtODE5MDA3ODQ0LDEyNDUwNzU4MjgsMT
M0NzY1NDE5MCwyMDc5OTEyMDc0LC0xMjA0NTA2NDg3LC0xNTkx
OTM5NDI5XX0=
-->