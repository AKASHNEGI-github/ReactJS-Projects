# Food Delivery Application
---

| Backend |
| Admin |
| Frontend |

---

## Backend

- server.js
```js
import express from "express";
import cors from "cors";
import { connectDB } from "./config/db.js";
import foodRouter from "./routes/foodRoute.js";
import userRouter from "./routes/userRoute.js";
import cartRouter from "./routes/cartRoute.js";
import orderRouter from "./routes/orderRoute.js";
import 'dotenv/config';

// Express
const app = express();
const port = 4000;

// Middleware
app.use(express.json());
app.use(cors());

// Database Connection
connectDB();

// API Endpoints
app.use("/api/food" ,foodRouter);
app.use("/images" ,express.static('uploads'));
app.use("/api/user", userRouter);
app.use("/api/cart", cartRouter);
app.use("/api/order", orderRouter);

app.get("/", (req, res) => {
    res.send("API Working");
});

app.listen(port, () => {
    console.log(`Server started on ${port}`);
});
```
### config

- config/db.js
```js
import mongoose from "mongoose";
// Connect Mongoose
export const connectDB = async () => {
    await mongoose.connect("mongodb://127.0.0.1:27017/bookDB")
    .then(() => {
        console.log("MongoDB Connected");
    })
    .catch((error) => {
        console.log(error);
    })
};
```

### models

- models/userModel.js
```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
    name: {type:String,required:true},
    email: {type:String,required:true,unique:true},
    password: {type:String,required:true},
    cartData: {type:Object,default:{}}
}, {minimize:false});

const userModel = mongoose.models.user || mongoose.model("user",userSchema);

export default userModel;
```

- models/foodModel.js
```js
import mongoose from "mongoose";

const foodSchema = new mongoose.Schema({
    name: {type:String,required:true},
    description: {type:String,required:true},
    price: {type:Number,required:true},
    category: {type:String,required:true},
    image: {type:String,required:true}
});

export const foodModel = mongoose.models.food || mongoose.model("food",foodSchema);
```

- models/orderModel.js
```js
import mongoose from "mongoose";

const orderSchema = new mongoose.Schema({
    userId: {type: String, required: true },
    items: {type: Array, required: true },
    amount: {type: Number, required: true },
    address: {type: Object, required: true },
    status: {type: String, default: "Food Processing"},
    date: {type: Date, default: Date.now()},
    payment: {type: Boolean, default: false }
});

const orderModel = mongoose.models.order || mongoose.model('order', orderSchema);

export default orderModel;
```

### routes

- routes/userRoute.js
```js
import express from "express";
import { registerUser, loginUser } from "../controllers/userController.js";

const userRouter = express.Router();

userRouter.post("/register", registerUser);
userRouter.post("/login", loginUser);

export default userRouter;
```

- routes/foodRoute.js
```js
import express from "express";
import { addFood, listFood, removeFood } from "../controllers/foodController.js";
import multer from "multer";

const foodRouter = express.Router();
// Image Storage Engine
const storage = multer.diskStorage({
    destination: "uploads",
    filename: (req, file, cb) => {
        return cb(null, `${Date.now()} ${file.originalname}`);
    }
});

const upload = multer({storage:storage});

foodRouter.post("/add", upload.single("image"), addFood);
foodRouter.get("/list", listFood);
foodRouter.post("/remove", removeFood);

export default foodRouter;
```

- routes/cartRoute.js
```js
import express from 'express';
import { addToCart, getCart, removeFromCart } from '../controllers/cartController.js';
import authMiddleware from '../middleware/auth.js';

const cartRouter = express.Router();

cartRouter.post('/add', authMiddleware, addToCart);
cartRouter.post('/remove', authMiddleware, removeFromCart);
cartRouter.post('/get', authMiddleware, getCart);

export default cartRouter;
```

- routes/orderRoute.js
```js
import express from "express";
import authMiddleware from "../middleware/auth.js";
import { listOrders, placeOrder, verifyOrder, updateStatus, userOrders } from "../controllers/orderController.js";

const orderRouter = express.Router();

orderRouter.post('/place', authMiddleware, placeOrder);
orderRouter.post('/verify', verifyOrder);
orderRouter.post('/userorders', authMiddleware, userOrders);
orderRouter.get('/list', listOrders);
orderRouter.post('/status', updateStatus);

export default orderRouter;
```

### middleware

- middleware/auth.js
```js
import jwt from 'jsonwebtoken';

const authMiddleware = async (req, res, next) => {
    const {token} = req.headers;
    if(!token) {
        return res.json({success: false, message: "Authorization token required"});
    }

    try {
        const token_decode = jwt.verify(token, process.env.JWT_SECRET);
        req.body.userId = token_decode.id;
        next();
    } catch (error) {
        return res.json({success: false, message: "Invalid Token"});
    }
}

export default authMiddleware;
```

### controllers

- controllers/userController.js
```js
import userModel from "../models/userModel.js";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken"; 
import validator from "validator"


// Login User
const loginUser = async (req, res) => {
    const {email, password} = req.body;
    try{
        const user = await userModel.findOne({email});
        if(!user){
            return res.json({success:false, message:"User Not Exists"});
        }
        const isMatch = await bcrypt.compare(password, user.password);
        if(!isMatch){
            return res.json({success:false, message:"Invalid Credentials"});
        }

        const token = createToken(user._id);
        res.json({success:true, token});
    }
    catch(error){
        console.log(error);
        res.json({success:false, message:"Error"});
    }
}

// Create Token
const createToken = (id) => {
    return jwt.sign({id}, process.env.JWT_SECRET);
}

// Register User
const registerUser = async (req, res) => {
    const {name, email, password} = req.body;
    try{
        // If User Already Exists
        const exists = await userModel.findOne({email});
        if(exists){
            return res.json({success:false, message:"User already registered"});
        } 
        // Validate Email
        if(!validator.isEmail(email)){
            return res.json({success:false, message:"Please, Enter Valid Email"});
        }
        if(password.length < 8){
            return res.json({success:false, message:"Password must be at least 8 characters"});
        }
        // Hash Password
        const salt = await bcrypt.genSalt(10);
        const hashedPassword = await bcrypt.hash(password, salt);

        const newUser = new userModel({
            name:name,
            email:email,
            password:hashedPassword
        });

        const user = await newUser.save();
        const token = createToken(user._id);
        res.json({success:true, token});
    }
    catch(error){
        console.log(error);
        res.json({success:false, message:"Error in Registration"});
    }
}

export {loginUser, registerUser};
```

- controllers/foodController.js
```js
import { foodModel } from "../models/foodModel.js";
import fs from "fs";

// Add Food Item
const addFood = async (req, res) => {
    let image_filename = `${req.file.filename}`;

    const food = new foodModel({
        name: req.body.name,
        description: req.body.description,
        price: req.body.price,
        category: req.body.category,
        image: image_filename
        //image: req.body.image
    })
    try{
        await food.save();
        res.json({success:true, message:"Food Added"});
    } catch(error){
        console.log(error);
        res.json({success:false, message:"Error"});
    }
};

// All Food List
const listFood = async (req, res) => {
    try{
        const foods = await foodModel.find({});
        res.json({success:true, data:foods});
    } catch(error){
        console.log(error);
        res.json({success:false, message:"Error"});
    }
}

// Remove Food Item
const removeFood = async (req, res) => {
    try{
        const food = await foodModel.findById(req.body.id);
        fs.unlink(`uploads/${food.image}`, () => {})

        await foodModel.findByIdAndDelete(req.body.id);
        res.json({success:true, message:"Food Removed"});
    } catch(error){
        console.log(error);
        res.json({success:false, message:"Error"});
    }
}

export {addFood, listFood, removeFood};
```

- controllers/cartController.js
```js
import userModel from "../models/userModel.js";

// Add Items to User Cart
const addToCart = async (req, res) => {
    try {
        let userData = await userModel.findById(req.body.userId);
        let cartData = await userData.cartData;
        if(!cartData[req.body.itemId]){
            cartData[req.body.itemId] = 1;
        }
        else{
            cartData[req.body.itemId] += 1;
        }
        await userModel.findByIdAndUpdate(req.body.userId, {cartData});
        res.json({success: true, message: "added to cart"});
    }
    catch (error) {
        res.json({success: false, message: "Error"});
    }
};

// Remove Items to User Cart
const removeFromCart = async (req, res) => {
    try {
        let userData = await userModel.findById(req.body.userId);
        let cartData = await userData.cartData;
        if(cartData[req.body.itemId] > 0){
            cartData[req.body.itemId] -= 1;
        }
        await userModel.findByIdAndUpdate(req.body.userId, {cartData});
        res.json({success: true, message: "removed from cart"});
    }
    catch (error) {
        res.json({success: false, message: "Error"});
    }
};

// Ferch User Cart Data
const getCart = async (req, res) => {
    try {
        let userData = await userModel.findById(req.body.userId);
        let cartData = await userData.cartData;
        res.json({success: true, cartData});
    }
    catch (error) {
        res.json({success: false, message: "Error"});
    }
};

export { addToCart, removeFromCart, getCart };
```

- controllers/orderController.js
```js
import orderModel from "../models/orderModel.js";
import userModel from "../models/userModel.js";
import Stripe from "stripe";
import dotenv from "dotenv";
dotenv.config();

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

// Place User Order From Frontend
const placeOrder = async (req, res) => {

    const frontend_url = "http://localhost:5173";

    try{
        const newOrder = new orderModel({
            userId: req.body.userId,
            items: req.body.items,
            amount: req.body.amount,
            address: req.body.address,
        });

        await newOrder.save();
        await userModel.findByIdAndUpdate(req.body.userId, {cartData: {}});

        const line_items = req.body.items.map((item) => ({
            price_data: {
                currency: 'inr',
                product_data: {name: item.name},
                unit_amount: item.price * 100 * 80,
            },
            quantity: item.quantity,
        }));

        line_items.push({
            price_data: {
                currency: 'inr',
                product_data: {name: "Delivery Charges"},
                unit_amount: 2 * 100 * 80,
            },
            quantity: 1,
        });

        const session = await stripe.checkout.sessions.create({
            line_items: line_items,
            mode: 'payment',
            success_url: `${frontend_url}/verify?success=true&orderId=${newOrder._id}`,
            cancel_url: `${frontend_url}/verify?success=false&orderId=${newOrder._id}`,
        });

        res.json({success: true, session_url: session.url});
    }
    // catch(error){
    //     res.json({success: false, message: "Error"});
    // }

    catch (error) {
    console.error("Stripe / Order Error:", error.message, error);
    res.json({ success: false, message: error.message });
    }

};

// Verify Order after Payment
const verifyOrder = async (req, res) => {
    const { orderId, success } = req.body;
    try {
        if (success == "true") {
            await orderModel.findByIdAndUpdate(orderId, {payment:true});
            res.json({ success: true, message: "Paid"});
        } 
        else {
            await orderModel.findByIdAndDelete(orderId);
            res.json({ success: false, message: "Payment failed" });
        }
    }
    catch (error) {
        res.json({ success: false, message: "Error" });
    }
};

// User Orders for User(Frontend)
const userOrders = async (req, res) => {
    try{
        const orders = await orderModel.find({userId: req.body.userId});
        res.json({success: true, data: orders});
    }
    catch(error){
        console.log(error);
        res.json({success: false, message: "Error"});
    }
};

// Listing orders for Admin-Pannel
const listOrders = async (req, res) => {
    try{
        const orders = await orderModel.find({});
        res.json({success: true, data: orders});
    }
    catch(error){
        res.json({success: false, message: "Error"});
    }
};

// API for Updating Order Status
const updateStatus = async (req, res) => {
    try{
        await orderModel.findByIdAndUpdate(req.body.orderId, {status: req.body.status});
        res.json({success: true, message: "Status Updated"});
    }
    catch(error){
        res.json({success: false, message: "Error"});
    }
};

export { placeOrder , verifyOrder , userOrders , listOrders , updateStatus};
```

---

## Admin

- main.jsx
```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'
import {BrowserRouter} from 'react-router-dom'

createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
)
```

- App.jsx
```jsx
import { useState } from 'react'
import Navbar from './components/Navbar/Navbar'
import Sidebar from './components/sidebar/Sidebar'
import { Route, Routes } from 'react-router-dom'
import Add from './pages/Add/Add'
import List from './pages/List/List'
import Orders from './pages/Orders/Orders'
import { ToastContainer } from 'react-toastify';
// import './App.css'

function App() {

  const url = "http://localhost:4000";

  return (
    <>
      <ToastContainer/>
      <Navbar/>
      <hr />
      <div className="app-content">
        <Sidebar/>
        <Routes>
          <Route path='/add' element={<Add url={url} />} />
          <Route path='/list' element={<List url={url} />} />
          <Route path='/orders' element={<Orders url={url} />} />
        </Routes>
      </div>
    </>
  )
}

export default App
```

### components

- components/Navbar.jsx
```jsx
import React from 'react'
import './Navbar.css'
import {assets} from '../../assets/assets'

const Navbar = () => {
  return (
    <div className='navbar'>
      <img className="logo" src={assets.logo} alt="" />
      <img className="profile" src={assets.profile_image} alt="" />
    </div>
  )
}

export default Navbar
```

- components/Sidebar.jsx
```jsx
import React from 'react'
import './Sidebar.css'
import { assets } from '../../assets/assets'
import { NavLink } from 'react-router-dom'

const Sidebar = () => {
  return (
    <div className='sidebar'>
      <div className="sidebar-options">
        <NavLink to='/add' className="sidebar-option">
            <img src={assets.add_icon} />
            <p>Add Items</p>
        </NavLink>
        <NavLink to='/list' className="sidebar-option">
            <img src={assets.order_icon} />
            <p>List Items</p>
        </NavLink>
        <NavLink to='/orders' className="sidebar-option">
            <img src={assets.order_icon} />
            <p>Orders</p>
        </NavLink>
      </div>
    </div>
  )
}

export default Sidebar
```

### pages

- pages/Add.jsx
```jsx
import React, { useState } from 'react'
import './Add.css'
import { assets } from '../../assets/assets'
import axios from 'axios'
import { toast } from 'react-toastify'

const Add = ({url}) => {

    //const url = "http://localhost:4000";
    const [image, setImage] = useState(false);
    const [data, setData] = useState({
        name: "",
        description: "",
        price: "",
        category: "Salad",
    });

    const onChangeHandler = (event) => {
        const name = event.target.name;
        const value = event.target.value;
        setData(data => ({...data, [name]:value}));
    };

    const onSubmitHandler = async (event) => {
        event.preventDefault();
        const formData = new FormData();
        formData.append('name', data.name);
        formData.append('description', data.description);
        formData.append('price', Number(data.price));
        formData.append('category', data.category);
        formData.append('image', image);

        const response = await axios.post(`${url}/api/food/add`, formData);
        if(response.data.success){
            setData({
                name: "",
                description: "",
                price: "",
                category: "Salad",
            });
            setImage(false);
            toast.success(response.data.message);
        }
        else{
            toast.error(response.data.message);
        }
    }

  return (
    <div className='add'>
        <form className="flex-col" onSubmit={onSubmitHandler}>
            <div className="add-img-upload flex-col">
                <p>Upload Image</p>
                <label htmlFor="image">
                    <img src={image?URL.createObjectURL(image):assets.upload_area} alt="" />
                </label>
                <input onChange={(e) => {setImage(e.target.files[0])}} type="file" id='image' hidden required />
            </div>
            <div className="add-product-name flex-col">
                <p>Product Name</p>
                <input onChange={onChangeHandler} value={data.name} type='text' name='name' placeholder='Type here' />
            </div>
            <div className="add-product-description flex-col">
                <p>Product Description</p>
                <textarea onChange={onChangeHandler} value={data.description} name='description' rows="6" placeholder='Write content here' required ></textarea>
            </div>
            <div className="add-category-price">
                <div className="add-category flex-col">
                    <p>Product Category</p>
                    <select onChange={onChangeHandler} name="category">
                        <option value="Salad">Salad</option>
                        <option value="Rolls">Rolls</option>
                        <option value="Deserts">Deserts</option>
                        <option value="Sandwich">Sandwich</option>
                        <option value="Cake">Cake</option>
                        <option value="Pure Veg">Pure Veg</option>
                        <option value="Pasta">Pasta</option>
                        <option value="Noodles">Noodles</option>
                    </select>
                </div>
                <div className="add-price flex-col">
                    <p>Product Price</p>
                    <input onChange={onChangeHandler} value={data.price} type='Number' name='price' placeholder='$20' />
                </div>
            </div>
            <button type='submit' className='add-btn'>ADD</button>
        </form>
    </div>
  )
}

export default Add
```

- pages/List.jsx
```jsx
import React, { useState, useEffect } from 'react'
import './List.css'
import axios from 'axios';
import { toast } from 'react-toastify';

const List = ({url}) => {

  //const url = "http://localhost:4000";
  const [list, setList] = useState([]);

  const fetchList = async () => {
    const response = await axios.get(`${url}/api/food/list`);
    console.log(response.data);
    if(response.data.success){
      setList(response.data.data);
    }
    else{
      toast.error("Error");
    }
  }

  const removeFood = async (foodId) => {
    //console.log(foodId);
    const response = await axios.post(`${url}/api/food/remove`, {id: foodId});
    await fetchList();
    if(response.data.success){
      toast.success(response.data.message);
    }
    else{
      toast.error("Error");
    }
  }

  useEffect(() => {
    fetchList();
  }, []);

  return (
    <div className='list add flex-col'>
      <p>All Food List</p>
      <div className="list-table">
        <div className="list-table-format title">
          <b>Image</b>
          <b>name</b>
          <b>Category</b>
          <b>Price</b>
          <b>Action</b>
        </div>
        {list.map((item,index) => {
          return (
            <div key={index} className="list-table-format">
              <img src={`${url}/images/` + item.image} alt="" />
              <p>{item.name}</p>
              <p>{item.category}</p>
              <p>${item.price}</p>
              <p onClick={() => {removeFood(item._id)}} className='cursor'>X</p>
            </div>
          )
        })}
      </div>
    </div>
  )
}

export default List
```

- pages/Orders.jsx
```jsx
import React from 'react'
import './Orders.css'
import axios from 'axios'
import { useEffect, useState } from 'react'
import { toast } from 'react-toastify'
import { assets } from '../../assets/assets'


const Orders = ({url}) => {

  const [orders, setOrders] = useState([]);

  const fetchAllOrders = async () => {
    const response = await axios.get(url + "/api/order/list");
    if(response.data.success)
    {
      setOrders(response.data.data);
    }
    else{
      toast.error("Error");
    }
  };

  const statusHandler = async (event, orderId) => {
    const response = await axios.post(url + "/api/order/status", {orderId,status:event.target.value});
    if(response.data.success)
    {
      await fetchAllOrders();
    }
  };

  useEffect(() => {
    fetchAllOrders();
  }, []);

  return (
    <div className='order add'>
      <h3>Order Page</h3>
      <div className="order-list">
        {orders.map((order, index) => (
          <div key={index} className='order-item'>
            <img src={assets.parcel_icon} alt=''/>
            <div>
              <p className="order-item-food">
                {order.items.map((item, index) => {
                  if(index === order.items.length-1)
                  {
                    return item.name + " X " + item.quantity;
                  }
                  else{
                    return item.name + " X " + item.quantity + " , ";
                  }
                })}
              </p>
              <p className='order-item-name'>{order.address.firstName + " " + order.address.lastName}</p>
              <div className="order-item-address">
                <p>{order.address.street + ","}</p>
                <p>{order.address.city + " , " + order.address.state + " , " + order.address.country + " , " + order.address.zipcode}</p>
              </div>
              <p className='order-item-phone'>{order.address.phone}</p>
            </div>
            <p>Items : {order.items.length}</p>
            <p>${order.amount}</p>
            <select onChange={(event) => statusHandler(event, order._id)} value={order.status}>
                <option value="Food Processing">Food Processing</option>
                <option value="Out for Delivery">Out for Delivery</option>
                <option value="Delivered">Delivered</option>
            </select>
          </div>
        ))}
      </div>
    </div>
  )
}

export default Orders
```

---

## Frontend

- main.jsx
```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'
import { BrowserRouter } from 'react-router-dom'
import StoreContextProvider from './components/context/StoreContext.jsx'

ReactDOM.createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <StoreContextProvider>
      <App />
    </StoreContextProvider>
  </BrowserRouter>
)
```

- App.jsx
```jsx
import React, { useState } from "react";
import Navbar from "./components/Navbar/Navbar";
import { Route, Routes } from "react-router-dom";
import Home from "./pages/Home/Home";
import Cart from "./pages/Cart/Cart";
import PlaceOrder from "./pages/PlaceOrder/PlaceOrder";
import Footer from "./components/Footer/Footer";
import LoginPopup from "./components/LoginPopup/LoginPopup";
import MyOrders from "./pages/MyOrders/MyOrders";
import Verify from "./pages/Verify/Verify";

const App = () => {

  const [showLogin , setShowLogin] = useState(false);

  return (
    <>
      {showLogin?<LoginPopup setShowLogin={setShowLogin} />:<></>}
      <div className="app">
        <Navbar setShowLogin={setShowLogin} />
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/cart" element={<Cart />} />
          <Route path="/order" element={<PlaceOrder />} />
          <Route path="/verify" element={<Verify />} />
          <Route path="/myorders" element={<MyOrders />} />
        </Routes>
      </div>
      <Footer />
    </>
  );
};

export default App;
```

### components

- components/StoreContext.jsx
```jsx
import axios from "axios";
import { createContext, useEffect, useState } from "react";
//import { food_list } from "../../assets/assets";

export const StoreContext = createContext(null)

const StoreContextProvider = (props) => {

    // Add To Cart 
    const [cartItems , setCartItems] = useState({});
    const url = "http://localhost:4000";
    const [token, setToken] = useState("");
    const [food_list, setFoodList] = useState([]);
    
    const addToCart = async (itemId) => {
        if(!cartItems[itemId])
        {
            setCartItems((prev) => ({...prev , [itemId] : 1}))
        }
        else
        {
            setCartItems((prev) => ({...prev , [itemId] : prev[itemId]+1}))
        }

        if(token){
            await axios.post(url + "/api/cart/add", {itemId} , {headers: {token}});
        }
    }

    // Remove From Cart
    const removeFromCart = async (itemId) => {
        setCartItems((prev) => ({...prev , [itemId] : prev[itemId]-1}));

        if(token){
            await axios.post(url + "/api/cart/remove", {itemId} , {headers: {token}});
        }
    }

    const getTotalCartAmount = () => {
        let totalAmount = 0;
        for(const item in cartItems)
        {
            if(cartItems[item]> 0)
            {
                let itemInfo = food_list.find((product) => product._id === item);
                totalAmount = totalAmount + (itemInfo.price * cartItems[item]);
            }
        }
        return totalAmount;
    }

    const fetchFoodList = async () => {
        const response = await axios.get(url + "/api/food/list");
        setFoodList(response.data.data);
    }

    const loadCartData = async (token) => {
        const response = await axios.post(url + "/api/cart/get" , {} , {headers: {token}});
        setCartItems(response.data.cartData);
    }

    useEffect(() => {
        if(localStorage.getItem("token")){
            setToken(localStorage.getItem("token"));
        }
        async function loadData() {
            await fetchFoodList();
            if(localStorage.getItem("token")){
                setToken(localStorage.getItem("token"));
                await loadCartData(localStorage.getItem("token"));
            }
        }
        loadData();
    }, []);

    const contextValue = {food_list , cartItems , setCartItems , addToCart , removeFromCart , getTotalCartAmount , url , token , setToken};

    return (
        <StoreContext.Provider value={contextValue}>
            {props.children}
        </StoreContext.Provider>
    )
}

export default StoreContextProvider
```

- components/Login.jsx
```jsx
import React, { useContext } from 'react'
import'./LoginPopup.css'
import { useState } from 'react'
import { assets } from '../../assets/assets'
import { StoreContext } from '../context/StoreContext'
import axios from 'axios'

const LoginPopup = ({setShowLogin}) => {

  const {url , setToken} = useContext(StoreContext);

    const [currState , setCurrState] = useState("Log-In");
    const [data, setData] = useState({
        name: "",
        email: "",
        password: ""
    });

    const onChangeHandler = (event) => {
      const name = event.target.name;
      const value = event.target.value;
      setData(data => ({...data, [name]: value}));
    };

    const onLogin = async (event) => {
      event.preventDefault();
      let newUrl = url;
      if(currState === "Log-In"){
        newUrl = newUrl + "/api/user/login";
      }
      else{
        newUrl = newUrl + "/api/user/register";
      }

      const response = await axios.post(newUrl, data);

      if(response.data.success){
        setToken(response.data.token);
        localStorage.setItem("token", response.data.token);
        setShowLogin(false);
      }
      else{
        alert(response.data.message);
      }
    };

  return (
    <div className='login-popup'>
      <form onSubmit={onLogin} className="login-popup-container">
        <div className="login-popup-title">
            <h2>{currState}</h2>
            <img onClick={() => setShowLogin(false)} src={assets.cross_icon} alt=''/>
        </div>
        <div className="login-popup-inputs">
            {currState === "Log-In"?<></>:<input name='name' value={data.name} onChange={onChangeHandler} type='text' placeholder='Enter Name' required />}
            <input name='email' value={data.email} onChange={onChangeHandler} type='email' placeholder='Enter Email' required />
            <input name='password' value={data.password} onChange={onChangeHandler} type='password' placeholder='Enter Password' required />
        </div>
        <button type='submit'>{currState === "Sign-Up"?"Create Account":"Log-In"}</button>
        <div className="login-popup-condition">
            <input type='checkbox' required />
            <p>By continuing, I agree to the terms of use & privacy policy.</p>
        </div>
        {currState === "Log-In"
            ?<p>Create a New Account ? <span onClick={() => setCurrState("Sign-Up")}>Click Here</span></p>
            :<p>Already have an Account ? <span onClick={() => setCurrState("Log-In")}>Log-In Here</span></p>
        }
      </form>
    </div>
  )
}

export default LoginPopup
```

- components/Navbar.jsx
```jsx
import React, { useContext, useState } from 'react'
import './Navbar.css'
import { assets } from '../../assets/assets'
import { Link, useNavigate } from 'react-router-dom'
import { StoreContext } from '../context/StoreContext'

const Navbar = ({setShowLogin}) => {

  const [menu , setMenu] = useState("Home");

  const { getTotalCartAmount , token , setToken } = useContext(StoreContext);

  const navigate = useNavigate();

  const logout = () => {
    localStorage.removeItem("token");
    setToken("");
    navigate("/");
  };

  return (
    <div className='navbar'>
      <Link to='/'><img src={assets.logo} alt="" className="logo" /></Link>
      <ul className="navbar-menu">
        <Link to='/' onClick={() => setMenu("Home")} className={menu === "Home"?"active":""}>Home</Link>
        <a href='#explore-menu' onClick={() => setMenu("Menu")} className={menu === "Menu"?"active":""}>Menu</a>
        <a href='#app-download' onClick={() => setMenu("Mobile-App")} className={menu === "Mobile-App"?"active":""}>Mobile-App</a>
        <a href='#footer' onClick={() => setMenu("Contact-Us")} className={menu === "Contact-Us"?"active":""}>Contact-Us</a>
      </ul>
      <div className="navbar-right">
        <img src={assets.search_icon} alt="" />
        <div className="navbar-search-icon">
            <Link to='/cart'><img src={assets.basket_icon} alt="" /></Link>
            <div className={getTotalCartAmount() === 0?"":"dot"}></div>
        </div>
        {!token?<button onClick={() => setShowLogin(true)}>Sign-In</button> 
        :<div className='navbar-profile'>
          <img src={assets.profile_icon} alt=''/>
          <ul className="nav-profile-dropdown">
            <li onClick={() => navigate('/myorders')}><img src={assets.bag_icon} alt=''/><p>Orders</p></li>
            <hr />
            <li onClick={logout}><img src={assets.logout_icon} alt=''/><p>Logout</p></li>
          </ul>
        </div>}
      </div>
    </div>
  )
}

export default Navbar
```

- components/Header.jsx
```jsx
import React from 'react'
import './Header.css'

const Header = () => {
  return (
    <div className='header'>
      <div className='header-contents'>
        <h2>Order your favourite food here</h2>
        <p>Choose from a diverse menu featuring a delectable array of dishes crafted with the finest ingredients and culinary expertise. Our mission is to satisfy your craving and elevate your dining experience, one delicious meal at a time.</p>
        <button>View Menu</button>
      </div>
    </div>
  )
}

export default Header
```

- components/ExploreMenu.jsx
```jsx
import React from 'react'
import './ExploreMenu.css'
import { menu_list } from '../../assets/assets'

const ExploreMenu = ({category , setCategory}) => {
  return (
    <div className='explore-menu' id='explore-menu'>
      <h1>Explore our menu</h1>
      <p className='explore-menu-text'>Choose from a diverse menu featuring a delectable array of dishes crafted with the finest ingredients and culinary expertise. Our mission is to satisfy your craving and elevate your dining experience, one delicious meal at a time.</p>
      <div className="explore-menu-list">
        {menu_list.map((item , index) => {
            return (
                <div onClick={() => setCategory(prev => prev === item.menu_name? "All":item.menu_name)} key={index} className="explore-menu-list-item">
                    <img className={category === item.menu_name? "active":""} src={item.menu_image} alt='' />
                    <p>{item.menu_name}</p>
                </div>
            )
        })}
      </div>
      <hr />
    </div>
  )
}

export default ExploreMenu
```

- components/FoodDisplay.jsx
```jsx
import React, { useContext } from 'react'
import './FoodDisplay.css'
import { StoreContext } from '../context/StoreContext'
import FoodItem from '../FoodItem/FoodItem'

const FoodDisplay = ({category}) => {

    const {food_list} = useContext(StoreContext)

  return (
    <div className='food-display' id='food-display'>
      <h2>Top Dishes near you</h2>
      <div className="food-display-list">
        {food_list.map((item , index) => {
            if(category === "All" || category === item.category)
            {
              return (
                <FoodItem key={index} id={item._id} name={item.name} description={item.description} price={item.price} image={item.image} />
              )
            }
        })}
      </div>
    </div>
  )
}

export default FoodDisplay
```

- components/FoodItem.jsx
```jsx
import React, { useContext } from 'react'
import './FoodItem.css'
import { assets } from '../../assets/assets'
import { useState } from 'react'
import { StoreContext } from '../context/StoreContext'

const FoodItem = ({id , name , price , description , image}) => {

  const { cartItems , addToCart , removeFromCart , url } = useContext(StoreContext);

  return (
    <div className='food-item'>
        <div className="food-item-img-container">
            <img className='food-item-image' src={url + "/images/" + image} alt='' />
            {
                !cartItems[id]
                    ?<img className='add' onClick={() => addToCart(id)} src={assets.add_icon_white} alt='' />
                    :<div className="food-item-counter">
                        <img onClick={() => removeFromCart(id)} src={assets.remove_icon_red} />
                        <p>{cartItems[id]}</p>
                        <img onClick={() => addToCart(id)} src={assets.add_icon_green} />
                    </div>
            }
        </div>
        <div className="food-item-info">
            <div className="food-item-name-rating">
                <p>{name}</p>
                <img src={assets.rating_starts} />
            </div>
            <p className="food-item-desc">{description}</p>
            <p className="food-item-price">${price}</p>
        </div>
    </div>
  )
}

export default FoodItem
```

- components/AppDownload.jsx
```jsx
import React from 'react'
import './AppDownload.css'
import { assets } from '../../assets/assets'

const AppDownload = () => {
  return (
    <div className='app-download' id='app-download'>
      <p>For Beter Experience Download <br /> Tomato App</p>
      <div className="app-download-platforms">
        <img src={assets.play_store} alt="" />
        <img src={assets.app_store} alt="" />
      </div>
    </div>
  )
}

export default AppDownload
```

- components/Footer.jsx
```jsx
import React from 'react'
import './Footer.css'
import { assets } from '../../assets/assets'

const Footer = () => {
  return (
    <div className='footer' id='footer'>
      <div className="footer-content">
        <div className="footer-content-left">
            <img src={assets.logo} alt="" />
            <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Earum quisquam, reiciendis sed explicabo rerum adipisci ipsa nemo blanditiis aut possimus amet perferendis iste atque iure architecto, necessitatibus deleniti error accusantium!</p>
            <div className="footer-social-icons">
                <img src={assets.facebook_icon} alt="" />
                <img src={assets.twitter_icon} alt="" />
                <img src={assets.linkedin_icon} alt="" />
            </div>
        </div>
        <div className="footer-content-center">
            <h2>COMPANY</h2>
            <ul>
                <li>Home</li>
                <li>About Us</li>
                <li>Delivery</li>
                <li>Privacy Policy</li>
            </ul>
        </div>
        <div className="footer-content-right">
            <h2>GET IN TOUCH</h2>
            <ul>
                <li>+91 987654321</li>
                <li>contact@tomato.com</li>
            </ul>
        </div>
      </div>
      <hr />
      <p className="footer-copyright">Copyright 2025 @ Tomato.com - All Right Reserved.</p>
    </div>
  )
}

export default Footer
```

### pages

- pages/Home.jsx
```jsx
import React, { useState } from 'react'
import './Home.css'
import Header from '../../components/Header/Header'
import ExploreMenu from '../../components/ExploreMenu/ExploreMenu'
import FoodDisplay from '../../components/FoodDisplay/FoodDisplay'
import AppDownload from '../../components/AppDownload/AppDownload'

const Home = () => {

  const [category , setCategory] = useState("All");

  return (
    <div>
      <Header />
      <ExploreMenu category={category} setCategory={setCategory} />
      <FoodDisplay category={category} />
      <AppDownload />
    </div>
  )
}

export default Home
```

- pages/Cart.jsx
```jsx
import React, { useContext } from "react";
import "./Cart.css";
import { StoreContext } from "../../components/context/StoreContext";
import { useNavigate } from "react-router-dom";

const Cart = () => {
  const { food_list, cartItems, removeFromCart , getTotalCartAmount , url } = useContext(StoreContext);

  const navigate = useNavigate();

  return (
    <div className="cart">
      <div className="cart-items">
        <div className="cart-items-title">
          <p>Items</p>
          <p>Title</p>
          <p>Price</p>
          <p>Quantity</p>
          <p>Total</p>
          <p>Remove</p>
        </div>
        <br />
        <hr />
        {food_list.map((item, index) => {
          if (cartItems[item._id] > 0) {
            return (
              <>
                <div className="cart-items-title cart-items-item">
                  <img src={url + "/images/" + item.image} alt="" />
                  <p>{item.name}</p>
                  <p>{item.price}</p>
                  <p>{cartItems[item._id]}</p>
                  <p>${item.price * cartItems[item._id]}</p>
                  <p onClick={() => removeFromCart(item._id)} className="cross">X</p>
                </div>
                <hr />
              </>
            );
          }
        })}
      </div>
      <div className="cart-bottom">
        <div className="cart-total">
          <h2>Cart Totals</h2>
          <div>
            <div className="cart-total-details">
              <p>Subtotal</p>
              <p>${getTotalCartAmount()}</p>
            </div>
            <hr />
            <div className="cart-total-details">
              <p>Delivery Fee</p>
              <p>${getTotalCartAmount() === 0?0:2}</p>
            </div>
            <hr />
            <div className="cart-total-details">
              <b>Total</b>
              <b>${getTotalCartAmount() === 0?0:getTotalCartAmount() + 2}</b>
            </div>
          </div>
          <button onClick={() => navigate('/order')}>Proceed to Checkout</button>
        </div>
        <div className="cart-promocode">
          <div>
            <p>If you have Promo-Code, Enter it here</p>
            <div className="cart-promocode-input">
              <input type="text" placeholder="Enter Promo-Code" />
              <button>Submit</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default Cart;
```

- pages/PlaceOrder.jsx
```jsx
import React, { useContext, useEffect, useState } from "react";
import "./PlaceOrder.css";
import { StoreContext } from "../../components/context/StoreContext";
import axios from "axios";
import { useNavigate } from "react-router-dom";

const PlaceOrder = () => {

  const { getTotalCartAmount , token , food_list , cartItems , url } = useContext(StoreContext);

  const [data , setData] = useState({
    firstName :"",
    lastName :"",
    email: "",
    street:"",
    city:"",
    state:"",
    zipcode:"",
    country:"",
    phone:""
  });

  const onChangeHandler = (event) => {
    const name = event.target.name;
    const value = event.target.value;
    setData(data => ({...data , [name]: value}));
  };

  const placeOrder = async (event) => {
    event.preventDefault();
    let orderItems = [];
    food_list.map((item) => {
      if(cartItems[item._id] > 0)
      {
        let itemInfo = item;
        itemInfo["quantity"] = cartItems[item._id];
        orderItems.push(itemInfo);
      }
    });

    let orderData = {
      address: data,
      items: orderItems,
      amount: getTotalCartAmount() + 2,
    };

    let response = await axios.post(url + '/api/order/place' , orderData , {headers:{token:localStorage.getItem("token")}});

    if(response.data.success)
    {
      const {session_url} = response.data;
      window.location.replace(session_url);
    }
    else{
      console.error(response);
      alert("Error");
    }
  };

  const navigate = useNavigate();

  useEffect(() => {
    if(!token)
    {
      navigate('/cart');
    }
    else if(getTotalCartAmount() === 0)
    {
      navigate('/cart');
    }
  }, [token]);

  return (
    <>
      <form onSubmit={placeOrder} className="place-order">
        <div className="place-order-left">
          <p className="title">Delivery Information</p>
          <div className="multi-fields">
            <input required name="firstName" value={data.firstName} onChange={onChangeHandler} type="text" placeholder="First Name" />
            <input required name="lastName" value={data.lastName} onChange={onChangeHandler} type="text" placeholder="Last Name" />
          </div>
          <input required name="email" value={data.email} onChange={onChangeHandler} type="email" placeholder="Email Address" />
          <input required name="street" value={data.street} onChange={onChangeHandler} type="text" placeholder="Street" />
          <div className="multi-fields">
            <input required name="city" value={data.city} onChange={onChangeHandler} type="text" placeholder="City" />
            <input required name="state" value={data.state} onChange={onChangeHandler} type="text" placeholder="State" />
          </div>
          <div className="multi-fields">
            <input required name="zipcode" value={data.zipcode} onChange={onChangeHandler} type="text" placeholder="Zip Code" />
            <input required name="country" value={data.country} onChange={onChangeHandler} type="text" placeholder="Country" />
          </div>
          <input required name="phone" value={data.phone} onChange={onChangeHandler} type="text" placeholder="Phone" />
        </div>
        <div className="place-order-right">
          <div className="cart-total">
            <h2>Cart Totals</h2>
            <div>
            <div className="cart-total-details">
              <p>Subtotal</p>
              <p>${getTotalCartAmount()}</p>
            </div>
            <hr />
            <div className="cart-total-details">
              <p>Delivery Fee</p>
              <p>${getTotalCartAmount() === 0?0:2}</p>
            </div>
            <hr />
            <div className="cart-total-details">
              <b>Total</b>
              <b>${getTotalCartAmount() === 0?0:getTotalCartAmount() + 2}</b>
            </div>
            </div>
            <button type="submit">Proceed to Pay</button>
          </div>
        </div>
      </form>
    </>
  );
};

export default PlaceOrder
```

- pages/Verify.jsx
```jsx
import React, { useContext, useEffect } from 'react'
import './Verify.css'
import { useNavigate, useSearchParams } from 'react-router-dom'
import { StoreContext } from '../../components/context/StoreContext';
import axios from 'axios';

const Verify = () => {

  const [searchParams, setSearchParams] = useSearchParams();
  const success = searchParams.get("success");
  const orderId = searchParams.get("orderId");
  const {url} = useContext(StoreContext);
  const navigate = useNavigate();

  const verifyPayment = async () => {
    const response = await axios.post(url + "/api/order/verify", {success, orderId});
    if(response.data.success)
    {
      navigate("/myorders");
    }
    else
    {
      navigate("/");
    }
  };

  useEffect(() => {
    verifyPayment();
  }, []);

  return (
    <div className='verify'>
      <div className="spinner"></div>
    </div>
  )
}

export default Verify
```

- pages/MyOrders.jsx
```jsx
import React, { useContext, useEffect, useState } from 'react'
import './MyOrders.css'
import { StoreContext } from '../../components/context/StoreContext'
import axios from 'axios';
import { assets } from '../../assets/assets';

const MyOrders = () => {

    const {url, token} = useContext(StoreContext);
    const [data, setData] = useState([]);

    const fetchOrders = async () => {
        const response = await axios.post(url + "/api/order/userorders", {}, {headers:{token}});
        setData(response.data.data);
    }

    useEffect(() => {
        if(token){
            fetchOrders();
        }
    }, [token]);

  return (
    <div className='my-orders'>
        <h2>My Orders</h2>
        <div className="container">
            {data.map((order, index) => {
                return(
                    <div key={index} className='my-orders-order'>
                        <img src={assets.parcel_icon} alt='' />
                        <p>
                            {order.items.map((item, index) => {
                                if(index === order.items.length-1){
                                    return item.name + " X " + item.quantity;
                                }
                                else{
                                    return item.name + " X " + item.quantity + " , ";
                                }
                            })}
                        </p>
                        <p>${order.amount}.00</p>
                        <p>Items: {order.items.length}</p>
                        <p><span>&#x25cf;</span> <b>{order.status}</b></p>
                        <button onClick={fetchOrders}>Track Order</button>
                    </div>
                )
            })}
        </div>
    </div>
  )
}

export default MyOrders
```

---


