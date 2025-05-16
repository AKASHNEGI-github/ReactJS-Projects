# Time & Date
```jsx
import { useEffect, useState } from 'react'
import './App.css'

function App() {
  const [date , setDate] = useState()
  const [time , setTime] = useState()

  useEffect(() => {
    setInterval(() => {
      const date = new Date();
      setDate(date.toLocaleDateString("de-DE"));
      setTime(date.toLocaleTimeString());
    } , 1000)
  } , [])

  const clock = {
            padding: "20px 50px",
            marginTop: "10px",
            fontSize: "30px",
            color: "red",
            backgroundColor: "yellow",
            borderRadius: "10px",
            border: "5px solid black"
          }


  return (
    <>
      <div>
        <h1>Time & Date</h1>
        <h2 style={clock}>Date : {date}</h2>
        <h2 style={clock}>Time : {time}</h2>
      </div>
    </>
  )
}

export default App

```
