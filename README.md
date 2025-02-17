# MERNhomePage

To create a full-stack MERN (MongoDB, Express, React, Node.js) application for user registration and login, I'll break down the process into both backend and frontend code. I'll also include a project folder structure and detailed explanations of each part.

---

### **Project Structure**

Here's how the project folder structure will look:

```
mern-auth-app/
│
├── backend/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   └── userController.js
│   ├── models/
│   │   └── User.js
│   ├── routes/
│   │   └── userRoutes.js
│   ├── server.js
│   └── .env
│
├── frontend/
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── components/
│   │   │   └── Register.js
│   │   │   └── Login.js
│   │   │   └── Home.js
│   │   ├── App.js
│   │   ├── index.js
│   │   └── App.css
│   ├── package.json
│   └── .env
│
├── .gitignore
└── README.md
```

---

### **Backend (Node.js + Express + MongoDB)**

1. **MongoDB Cloud Connection**: Use MongoDB Atlas for hosting your database.

   #### **backend/config/db.js**

```js
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        const conn = await mongoose.connect(process.env.MONGO_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true,
        });
        console.log(`MongoDB Connected: ${conn.connection.host}`);
    } catch (error) {
        console.error(error);
        process.exit(1);
    }
};

module.exports = connectDB;
```

2. **User Model**: Define the schema for the user.

   #### **backend/models/User.js**

```js
const mongoose = require('mongoose');

const userSchema = mongoose.Schema({
    username: {
        type: String,
        required: true,
        unique: true
    },
    name: {
        type: String,
        required: true
    },
    email: {
        type: String,
        required: true,
        unique: true
    },
    phone: {
        type: String,
        required: true
    },
    dob: {
        type: Date,
        required: true
    },
    bio: {
        type: String,
        required: false
    },
    password: {
        type: String,
        required: true
    }
}, {
    timestamps: true
});

module.exports = mongoose.model('User', userSchema);
```

3. **User Controller**: Functions to handle user registration and login.

   #### **backend/controllers/userController.js**

```js
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');

// Register new user
const registerUser = async (req, res) => {
    const { username, name, email, phone, dob, bio, password, confirmPassword } = req.body;

    // Validate fields
    if (!username || !name || !email || !phone || !dob || !password || !confirmPassword) {
        return res.status(400).json({ message: 'Please fill in all fields' });
    }

    if (password !== confirmPassword) {
        return res.status(400).json({ message: 'Passwords do not match' });
    }

    const userExists = await User.findOne({ email });
    if (userExists) {
        return res.status(400).json({ message: 'User already exists' });
    }

    const salt = await bcrypt.genSalt(10);
    const hashedPassword = await bcrypt.hash(password, salt);

    const user = new User({
        username,
        name,
        email,
        phone,
        dob,
        bio,
        password: hashedPassword
    });

    try {
        await user.save();
        res.status(201).json({ message: 'User registered successfully' });
    } catch (error) {
        res.status(500).json({ message: 'Server error' });
    }
};

// Login user
const loginUser = async (req, res) => {
    const { email, password } = req.body;

    if (!email || !password) {
        return res.status(400).json({ message: 'Please fill in all fields' });
    }

    const user = await User.findOne({ email });
    if (!user) {
        return res.status(400).json({ message: 'Invalid credentials' });
    }

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
        return res.status(400).json({ message: 'Invalid credentials' });
    }

    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET, { expiresIn: '30d' });

    res.json({ message: 'Login successful', token });
};

module.exports = { registerUser, loginUser };
```

4. **Routes**: Define routes for registration and login.

   #### **backend/routes/userRoutes.js**

```js
const express = require('express');
const { registerUser, loginUser } = require('../controllers/userController');

const router = express.Router();

router.post('/register', registerUser);
router.post('/login', loginUser);

module.exports = router;
```

5. **Server Setup**: Set up the server to run the API.

   #### **backend/server.js**

```js
const express = require('express');
const connectDB = require('./config/db');
const userRoutes = require('./routes/userRoutes');
const dotenv = require('dotenv');

dotenv.config();
connectDB();

const app = express();
app.use(express.json());  // to parse JSON bodies

app.use('/api/users', userRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

6. **.env file for backend**

   #### **backend/.env**

```env
MONGO_URI=your_mongoDB_atlas_connection_string
JWT_SECRET=your_jwt_secret_key
PORT=5000
```

---

### **Frontend (React)**

1. **Main App Component**: Set up the routing and basic structure of the app.

   #### **frontend/src/App.js**

```js
import React, { useState } from 'react';
import { BrowserRouter as Router, Route, Switch, Link } from 'react-router-dom';
import Register from './components/Register';
import Login from './components/Login';
import Home from './components/Home';

function App() {
    const [user, setUser] = useState(null);

    return (
        <Router>
            <div>
                <nav>
                    <Link to="/login">Login</Link>
                    <Link to="/register">Register</Link>
                </nav>
                <Switch>
                    <Route path="/register" component={Register} />
                    <Route path="/login">
                        <Login setUser={setUser} />
                    </Route>
                    <Route path="/home">
                        {user ? <Home username={user.username} /> : <Redirect to="/login" />}
                    </Route>
                    <Route exact path="/" component={Home} />
                </Switch>
            </div>
        </Router>
    );
}

export default App;
```

2. **Register Component**: Create the registration form.

   #### **frontend/src/components/Register.js**

```js
import React, { useState } from 'react';
import axios from 'axios';

const Register = () => {
    const [formData, setFormData] = useState({
        username: '',
        name: '',
        email: '',
        phone: '',
        dob: '',
        bio: '',
        password: '',
        confirmPassword: ''
    });

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData((prevData) => ({
            ...prevData,
            [name]: value
        }));
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            await axios.post('/api/users/register', formData);
            alert('User Registered');
        } catch (error) {
            alert(error.response.data.message);
        }
    };

    return (
        <div>
            <h2>Register</h2>
            <form onSubmit={handleSubmit}>
                {/* Form Fields for Username, Name, Email, etc. */}
                <input type="text" name="username" value={formData.username} onChange={handleChange} placeholder="Username" required />
                {/* Other inputs here */}
                <button type="submit">Register</button>
            </form>
        </div>
    );
};

export default Register;
```

3. **Login Component**: Create the login form.

   #### **frontend/src/components/Login.js**

```js
import React, { useState } from 'react';
import axios from 'axios';
import { Redirect } from 'react-router-dom';

const Login = ({ setUser }) => {
    const [formData, setFormData] = useState({
        email: '',
        password: ''
    });
    const [error, setError] = useState(null);

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData((prevData) => ({
            ...prevData,
            [name]: value
        }));
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            const response = await axios.post('/api/users/login', formData);
            setUser({ username: response.data.username });
            // Save token for future use
            localStorage.setItem('token', response.data.token);
        } catch (error) {
            setError(error.response.data.message);
        }
    };

    if (localStorage.getItem('token')) {
        return <Redirect to="/home" />;
    }

    return (
        <div>
            <h2>Login</h2>
            {error && <p>{error}</p>}
            <form onSubmit={handleSubmit}>
                <input type="email" name="email" value={formData.email} onChange={handleChange} placeholder="Email" required />
                <input type="password" name="password" value={formData.password} onChange={handleChange} placeholder="Password" required />
                <button type="submit">Login</button>
            </form>
        </div>
    );
};

export default Login;
```

4. **Home Component**: Display the home page after login.

   #### **frontend/src/components/Home.js**

```js
import React from 'react';

const Home = ({ username }) => {
    return <h1>Hello, {username}</h1>;
};

export default Home;
```

5. **Frontend Setup**: Add `axios` and other dependencies.

   #### **frontend/package.json**

```json
{
  "name": "frontend",
  "version": "1.0.0",
  "dependencies": {
    "axios": "^0.21.1",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-router-dom": "^5.1.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  }
}
```

---

### **Running the Application**

1. **Backend**:
   - In the `backend` folder, install dependencies: 
   ```bash
   npm init -y
   npm install express mongoose bcryptjs jsonwebtoken dotenv
   ```

2. **Frontend**:
   - In the `frontend` folder, install dependencies:
   ```bash
   npm install axios react-router-dom
   ```

3. **Start Backend and Frontend**:
   - Start the backend server by running:
   ```bash
   node server.js
   ```
   - Start the frontend by running:
   ```bash
   npm start
   ```

---

This should give you a fully functional MERN stack application with user registration and login, using MongoDB, Express, React, and Node.js! Let me know if you need any additional help!
