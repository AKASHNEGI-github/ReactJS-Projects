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

  const btn = {
    margin: "10px",
    border: "2px solid black",
  };

  return (
    <>
      <h1>React Counter App</h1>
      <div style={{fontSize:"30px"}}>Count : {count}</div>
      <div>
        <button style={btn} onClick={increaseCount}>+</button>
        <button style={btn} onClick={resetCount}>↻</button>
        <button style={btn} onClick={decreaseCount}>-</button>
      </div>
    </>
  )
}

export default App
```
