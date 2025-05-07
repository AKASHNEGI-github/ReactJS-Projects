# Password generator
```jsx
import { useCallback, useEffect, useRef, useState } from 'react'

function App() {
  // useState
  const [length , setLength] = useState(5);
  const [numbers , setNumbers] = useState(true);
  const [smallAlphabets , setSmallAlphabets] = useState(false);
  const [capitalAlphabets , setCapitalAlphabets] = useState(false);
  const [specialCaracters , setSpecialCharacters] = useState(false);
  const [password , setPassword] = useState("");

  // useRef
  const passwordReference = useRef(null);
  const copyPasswordToClipboard = useCallback(function(){
    passwordReference.current?.select();
    window.navigator.clipboard.writeText(password);
  } , [password]);

  // useCallback
  const passwordGenerator = useCallback(function(){
    let password = "";
    let str = "";
    if(numbers){
      str = str + "0123456789";
    }
    if(smallAlphabets){
      str = str + "abcdefghijklmnopqrstuvwxyz";
    }
    if(capitalAlphabets){
      str = str + "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    }
    if(specialCaracters){
      str = str + "`~@#$%^&*()-_=+";
    }

    for(let i=0 ; i<length ; i++){
      const randomIndex = Math.floor(Math.random() * str.length);
      password = password + str.charAt(randomIndex);
    }

    setPassword(password); // Set Password
  } , [length , numbers , smallAlphabets , capitalAlphabets , specialCaracters]);

  // useEffect
  useEffect(function(){
    passwordGenerator(); // Function Call - Password Generator
  } , [length , numbers , smallAlphabets , capitalAlphabets , specialCaracters , passwordGenerator]);

  return (
    <>
      <div>
        <input type="text"  value={password} placeholder="Password" readOnly ref={passwordReference}></input>
        <button onClick={copyPasswordToClipboard}>copy</button>
      </div>
      <div>
        <p>Length : {length}</p>
        <input type="range" min={1} max={10} value={length} onChange={(event) => {setLength(Number(event.target.value))}}/>
        <input type="checkbox" defaultChecked={numbers} onChange={() => {setNumbers((prev) => !prev)}}/>
        <label>Numbers</label>
        <input type="checkbox" defaultChecked={smallAlphabets} onChange={() => {setSmallAlphabets((prev) => !prev)}}/>
        <label>Small Alphabets</label>
        <input type="checkbox" defaultChecked={capitalAlphabets} onChange={() => {setCapitalAlphabets((prev) => !prev)}}/>
        <label>Capital Alphabets</label>
        <input type="checkbox" defaultChecked={specialCaracters} onChange={() => {setSpecialCharacters((prev) => !prev)}}/>
        <label>special Character</label>
      </div>
    </>
  )
}

export default App


```
