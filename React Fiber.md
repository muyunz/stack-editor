
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
    
    // ㄅㄚ
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // Children can be more than one argument, and those are transferred onto
  // the newly allocated props object.
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (__DEV__) {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }

  // Resolve default props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NDAwNTU2NzQsLTEzMjI4NjEwMCw1OD
k1NTY3NjgsLTEzNjkzMzMzNTAsLTE4NTgxNDAwMzgsMzAzNDU2
NTg2XX0=
-->