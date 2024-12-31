- 👋 Hi, I’m @Anu782-ops
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...class 8
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Anu782-ops/Anu782-ops is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import React from "react";
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";
import Login from "./components/Login";
import Notes from "./components/Notes";

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/notes" element={<Notes />} />
      </Routes>
    </Router>
  );
}

export default App;
import React, { useState } from "react";
import axios from "axios";

const Login = ({ setLoggedIn }) => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  const handleLogin = async () => {
    try {
      const response = await axios.post("http://localhost:5000/login", {
        username,
        password,
      });
      if (response.data.success) {
        setLoggedIn(true);
      } else {
        alert("Invalid credentials");
      }
    } catch (error) {
      console.error("Login error:", error);
    }
  };

  return (
    <div>
      <h1>Login</h1>
      <input
        type="text"
        placeholder="Username"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
      <input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button onClick={handleLogin}>Login</button>
    </div>
  );
};

export default Login;
mkdir secure-notes-backend
cd secure-notes-backend
npm init -y
npm install express mongoose bcrypt jsonwebtoken cors
const express = require("express");
const mongoose = require("mongoose");
const bcrypt = require("bcrypt");
const jwt = require("jsonwebtoken");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

const SECRET_KEY = "your_secret_key";

// MongoDB connection
mongoose.connect("mongodb://localhost:27017/secure-notes", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// User schema
const userSchema = new mongoose.Schema({
  username: String,
  password: String,
});
const User = mongoose.model("User", userSchema);

// Note schema
const noteSchema = new mongoose.Schema({
  content: String,
  owner: String,
  isPrivate: Boolean,
});
const Note = mongoose.model("Note", noteSchema);

// Routes
app.post("/login", async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });
  if (user && bcrypt.compareSync(password, user.password)) {
    const token = jwt.sign({ username }, SECRET_KEY, { expiresIn: "1h" });
    res.json({ success: true, token });
  } else {
    res.json({ success: false });
  }
});

app.post("/register", async (req, res) => {
  const { username, password } = req.body;
  const hashedPassword = bcrypt.hashSync(password, 10);
  await User.create({ username, password: hashedPassword });
  res.json({ success: true });
});

app.post("/notes", async (req, res) => {
  const { token, content, isPrivate } = req.body;
  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    const newNote = await Note.create({
      content,
      owner: decoded.username,
      isPrivate,
    });
    res.json({ success: true, note: newNote });
  } catch (error) {
    res.status(401).json({ success: false, message: "Unauthorized" });
  }
});

app.get("/notes", async (req, res) => {
  const { token } = req.headers;
  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    const notes = await Note.find({ owner: decoded.username });
    res.json({ success: true, notes });
  } catch (error) {
    res.status(401).json({ success: false, message: "Unauthorized" });
  }
});

// Start server
app.listen(5000, () => {
  console.log("Server running on http://localhost:5000");
});
