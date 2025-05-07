# CountDown
```jsx
import { useState , useRef } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import './App.css'

function App() {
  const [counter , setCounter] = useState(0);
  const [isRunning , setIsRunning] = useState(false);
  const intervalRef = useRef(null);

  const startTimer = () => {
    if(isRunning)
    {
      return;
    }
    setIsRunning(true);
    intervalRef.current = setInterval(function(){
      setCounter(prev => prev + 1);
    }, 1000);
  }

  const stopTimer = () => {
    clearInterval(intervalRef.current);
    setIsRunning(false);
  }

  const resetTimer = () => {
    clearInterval(intervalRef.current);
    setIsRunning(false);
    setCounter(0);
  }

  const btn = {
    margin: "20px",
    border: "2px solid black",
  };
  return (
    <>
      <div style={{fontSize:"30px"}}>{counter}</div>
      <button style={btn} onClick={startTimer}>Start</button>
      <button style={btn} onClick={resetTimer}>Reset</button>
      <button style={btn} onClick={stopTimer}>Stop</button>
    </>
  )
}

export default App
```
