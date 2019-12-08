
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

#### React Element > React Fiber
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIzNDgxNTM0MCw1ODk1NTY3NjgsLTEzNj
kzMzMzNTAsLTE4NTgxNDAwMzgsMzAzNDU2NTg2XX0=
-->