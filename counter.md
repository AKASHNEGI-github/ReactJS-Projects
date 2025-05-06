# Counter
```jsx
import { useState } from 'react'
import './App.css'

function App() {
  let [count , setCount] = useState(0);

  const increaseCount = () => {
    setCount(count = count + 1);
  }

  const decreaseCount = () => {
    setCount(count = count - 1);
  }

  const resetCount = () => {
    setCount(count = 0);
  }

  return (
    <>
      <div>Counter App</div>
      <div>Count : {count}</div>
      <div>
        <div onClick={increaseCount}>+</div>
        <div onClick={resetCount}>reset</div>
        <div onClick={decreaseCount}>-</div>
      </div>
    </>
  )
}

export default App

```
