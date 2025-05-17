# Form

- Form-1 Using useState()
```jsx
import { useState } from "react"

export const Form1 = () => {
    const [name , setName] = useState("")
    const [email , setEmail] = useState("")
    const [phone , setPhone] = useState("")

    const handleSubmitForm = (event) => {
        event.preventDefault();
        console.log("Form-1 Using useState()");
        console.log(`Name : ${name} & Email : ${email} & Phone : ${phone}`);
    }

    return(
        <>
            <form onSubmit={handleSubmitForm}>
                <h2>Form-1 Using useState</h2>
                <div>
                    <label>Name : </label>
                    <input type="text" placeholder="Enter Name"  name="name" value={name} onChange={(event) => {setName(event.target.value)}}></input>
                </div>
                <div>
                    <label>Email : </label>
                    <input type="email" placeholder="Enter Email" name="email" value={email} onChange={(event) => {setEmail(event.target.value)}}></input>
                </div>
                <div>
                    <label>Phone Number : </label>
                    <input type="tel" placeholder="Enter Phone Number" name="phone" value={phone} onChange={(event) => {setPhone(event.target.value)}}></input>
                </div>
                <div>
                    <button>Submit</button>
                </div>
                <h3>Name : {name} | Email : {email} | Phone : {phone}</h3>
            </form>
        </>
    )
}
```

- Form-2 Using Object
```jsx
import { useState } from "react"

export const Form2 = () => {
    const [user , setUser] = useState({
        name : "",
        email : "",
        phone : ""
    }) 

    const handleInputChange = (event) => {
        const {name , value} = event.target;
        setUser((prev) => ({...prev , [name] : value}))
    }

    const handleSubmitForm = (event) => {
        event.preventDefault();
        console.log("Form-2 Using Object")
        console.log(user);
    }

    return(
        <>
            <form onSubmit={handleSubmitForm}>
                <h2>Form-2 Using Object</h2>
                <div>
                    <label>Name : </label>
                    <input type="text" placeholder="Enter Name"  name="name" value={user.name} onChange={handleInputChange}></input>
                </div>
                <div>
                    <label>Email : </label>
                    <input type="email" placeholder="Enter Email" name="email" value={user.email} onChange={handleInputChange}></input>
                </div>
                <div>
                    <label>Phone Number : </label>
                    <input type="tel" placeholder="Enter Phone Number" name="phone" value={user.phone} onChange={handleInputChange}></input>
                </div>
                <div>
                    <button>Submit</button>
                </div>
                <h3>Name : {user.name} | Email : {user.email} | Phone : {user.phone}</h3>
            </form>
        </>
    )
}
```
