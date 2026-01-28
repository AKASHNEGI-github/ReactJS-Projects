# Accordion

```styles.css
/* General Accordion Styles */
.accordion {
    width: 100%;
    max-width: 400px;
    margin: auto;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #f9f9f9;
}

/* Accordion Item Styles */
.accordion-item {
    margin-bottom: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    background-color: #fff;
}

/* Accordion Title Button */
.accordion-title {
    width: 100%;
    padding: 15px;
    text-align: left;
    background-color: #f1f1f1;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    font-weight: bold;
    transition: background-color 0.3s ease;
}

.accordion-title:hover {
    background-color: #e0e0e0;
}

.accordion-title[aria-expanded="true"] {
    background-color: #d0d0d0;
}

.accordion-content {
    padding: 15px;
    background-color: #fafafa;
    border-top: 1px solid #ddd;
    font-size: 14px;
}

.accordion p {
    font-size: 16px;
    color: #666;
    text-align: center;
}

.right {
    float: right;
}
```

```Accordion.js
import React, { useState } from 'react';
import { FaChevronDown, FaChevronUp } from "react-icons/fa";
import './styles.css'

function Accordion({ items }) {
    const [openIndex, setOpenIndex] = useState(null);

    const handleToggle = (index) => {
        if (index === openIndex) {
            setOpenIndex(null);
        }
        else {
            setOpenIndex(index);
        }
    }

    return !items || (items.length === 0) ? "No items available" : (
        <div className="accordion">
            {items.map((item, index) => {
                return (
                    <div key={index} className="accordion-item">
                        <button onClick={() => handleToggle(index)} className="accordion-title">
                            {item.title}
                            {index === openIndex ? <FaChevronUp className="right" /> : <FaChevronDown className="right" />}
                        </button>
                        {
                            index === openIndex && <div className="accordion-content">
                                <p>{item.content}</p>
                            </div>
                        }
                    </div>
                );
            })}
        </div>
    );
}

export default Accordion;
```

```App.js
import Accordion from "./Accordion"

export default function App() {

  const items = [
  {
    title: "JavaScript Basics",
    content: "Learn variables, functions, and loops in JavaScript."
  },
  {
    title: "React.js Overview",
    content: "Understand components, state, and props in React."
  },
  {
    title: "Node.js",
    content: "Basics of server-side development with Node.js."
  },
  {
    title: "Full-Stack Development",
    content: "Build full-stack apps with React and Node.js."
  },
];

return <Accordion items={items}/>
}

```
