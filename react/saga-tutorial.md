> 在开始学习本教程之前，我们假定你已经能够熟练使用Generator、redux和react。

redux-saga 是一个在 React/Redux 应用中，可以优雅地处理 side effect(副作用：异步等)的库，你可以在一个地方创建sagas来专门处理side effect。
## 一个简单的例子
在store中引入中间件`redux-saga`
```javascript
//store
import { createStore, applyMiddleware } from 'redux';
import createSagaMiddleware, { runSaga } from 'redux-saga';
import reducer from './reducers';
import saga from './sagas';

const sagaMiddleware = createSagaMiddleware();
const store = createStore(reducer, applyMiddleware(sagaMiddleware));
sagaMiddleware.run(saga);

//sagas.js
function* helloSagas(){
  console.log('hello saga');
}

export default helloSagas
```
运行程序，就可以在控制台看到`hello saga`。现在让我们的saga开始捕获`action`，下面是一个简单的`Counter`组件,对于这种同步的action，我们无需借助saga，但是当我们点击按钮式，saga仍然能够监听到action的执行。运行下面的程序，当点击按钮时，在控制台会打印相应的action，但action并不是由saga来调用的。
```js
// component
import React, { Component } from 'react'
import {connect} from 'react-redux' 
import * as actions from '@/actions'
import { getSagaCounter } from '@/reducers'

class Counter extends Component {
  render() {
    const { increment, decrement } = this.props
    return (
      <div>
        <button onClick={increment}>increment</button>
        <button onClick={decrement}>decrement</button>
        <br />
        { this.props.counter }
      </div>
    )
  }
}

export default connect(
  (state) => ({counter: getSagaCounter(state)}),
  actions
)(Counter)

//reducer
import { combineReducers } from 'redux';

const sagaCounter = (state = 0, action) => {
  switch(action.type){
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

export default combineReducers({
  sagaCounter
})

export const getSagaCounter = state => state.sagaCounter

//action
export const increment = () => ({
  type: 'INCREMENT'
})

export const decrement = () => ({
  type: 'DECREMENT'
})

//saga
import {takeEvery} from 'redux-saga'
function* listenAction(action){
  console.log(action);
}

function* incrementSagas(){
  //在每次 dispatch `INCREMENT` action 時，执行listenAction。
  yield takeEvery('INCREMENT', listenAction)
  //在每次 dispatch `DECREMENT` action 時，执行listenAction。
  yield takeEvery('DECREMENT', listenAction)
}

export default incrementSagas;
```
现在我们对做一点小改动，定义一个新的`action`:`MY_INCREMENT`，这个`action`的作用就是执行原来的`INCREMENT`，我们新添加一个按钮，点击时`dispatch`这个`action`

```js
const { increment, decrement, myIncrement } = this.props
return (
  ...
  <button onClick={myIncrement}>myIncrement</button>
  ...
)
//action
export const myIncrement = () => ({
  type: 'MY_INCREMENT'
})
//sagas
import {takeEvery, put} from 'redux-saga/effects'
function* increment(action){
  yield put({type:'INCREMENT'})//dispatch
}

function* incrementSagas(){
  yield takeEvery('MY_INCREMENT', increment)
}

export default incrementSagas;
```
当saga拿到`MY_INCREMENT`时，执行`increment`函数，而`increment`函数就是`dispatch INCREMENT`，从而实现的counter的增加。此时我们就发现，如果要MY_INCREMENT延迟执行的话，只需要在`increment`函数中yeild一个延迟函数,当一个Promise在saga中yield时，saga将暂停执行，知道这个Promise被resolve，然后saga继续执行。
```js
const delay = ms => new Promise(resolve => setTimeout(resolve, ms))

function* increment(action){
  yield delay(1000) //延迟1s
  yield api.fetchSomeData() // 等待数据返回
  yield put({type:'INCREMENT'}) //上面的都完成后，dispatch `INCREMENT` action
}
```
现在回过头，看看我们最开始的helloSagas，如果此时要在加上这个saga该如何实现呢，在`redux-saga/effects`里面有一个all方法，只需要将多个saga放入all里面就可以了，因此很容易实现saga文件的拆分
```js
export default function* rootSaga() {
  yield all([
    helloSaga(),
    incrementSagas()
  ])
}

```
到这里，我们大致已经了解了saga的基本原理，后面的教程将详细介绍具体的细节。
