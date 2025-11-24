# Food Delivery Application
---



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


