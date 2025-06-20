# Quiz App

```Quiz.jsx
import { useRef, useState } from 'react'
import './Quiz.css'
import { data } from '../../assets/data'

function Quiz() {

    let [index , setIndex] = useState(0)
    let [lock , setLock] = useState(false)
    let [questions , setQuestions] = useState(data[index])
    let [score , setScore] = useState(0)
    let [result , setResult] = useState(false)

    let option1 = useRef(null)
    let option2 = useRef(null)
    let option3 = useRef(null)
    let option4 = useRef(null)

    let options = [option1 , option2 , option3 , option4]

    const checkAnswer = (e , ans) =>
    {
        if(lock === false)
        {
            if(questions.ans === ans)
            {
                e.target.classList.add("correct")
                setScore(++score)
            }
            else
            {  
                e.target.classList.add("wrong")
                options[questions.ans - 1].current.classList.add("correct")
            }
            setLock(true)
        }
    }

    const next = () =>
    {
        if(lock === true)
        {
            if(index === data.length - 1)
            {
                setResult(true)
                return 0
            }
            setIndex(++index)
            setQuestions(data[index])
            setLock(false)
            options.map((Option) =>
            {
                Option.current.classList.remove("wrong")
                Option.current.classList.remove("correct")
                return null
            })
        }
    }

    const reset = () =>
    {
        setIndex(0)
        setQuestions(data[0])
        setScore(0)
        setLock(false)
        setResult(false)
    }

    return (
      <>
        <div className="container">
            <h1>Quiz App</h1>
            <hr />
            {
                result? <></> : 
                <>
                <h2>{index + 1}. {questions.question}</h2>
                <ul>
                    <li ref={option1} onClick={(e) => {checkAnswer(e , 1)}} >{questions.option1}</li>
                    <li ref={option2} onClick={(e) => {checkAnswer(e , 2)}} >{questions.option2}</li>
                    <li ref={option3} onClick={(e) => {checkAnswer(e , 3)}} >{questions.option3}</li>
                    <li ref={option4} onClick={(e) => {checkAnswer(e , 4)}} >{questions.option4}</li>
                </ul>
                <button onClick={next}>Next</button>
                <div className="index">{index + 1} of {data.length} Questions</div>
                <h6>© Copyright 2024 <span class="akashnegi">Akash Negi</span>. All rights reserved.</h6>
                </>
            }
            {
                result? 
                <>
                <h2 className="score">Score : {score} out of {data.length}</h2>
                <button onClick={reset}>Reset</button>
                <h6>© Copyright 2024 <span class="akashnegi">Akash Negi</span>. All rights reserved.</h6>
                </> : <></>
            }
        </div>
      </>
    )
  }

  export default Quiz
```
