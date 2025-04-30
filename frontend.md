# FRONTEND INSTALLATION/SETUP

## Task 1: Project Setup

```bash
npm create vite@latest frontend -- --template react
# Choose react
# Choose javascript
cd frontend
npm install 
npm run dev
npm install axios react-router-dom jwt-decode
```

## Task 2: Project Structure Setup

1. Inside the frontend directory, delete CSS files in the src folder (App.css and Index.css)
2. Open src/app.jsx and replace all code with:

```jsx
import React from 'react';

function App() {
  return (
    <>
    
    </>
  );
}

export default App;
```

3. Go to main.jsx and remove `import './index.css'`
4. Create the following directory structure inside src:
   - `pages/`
   - `styles/`
   - `components/`
5. Create these files at the src level:
   - `constants.js`
   - `api.js`
6. Create an environment variable file `.env` in the frontend root

![image](https://github.com/user-attachments/assets/8750d23d-bb9b-40b8-acad-fe66b2654684)



## Task 3: Configuration Files

1. In constants.js, add:

```javascript
export const ACCESS_TOKEN = "access"
export const REFRESH_TOKEN = "refresh"
```

2. In api.js, add:

```javascript
import axios from "axios"
import { ACCESS_TOKEN } from "./constants"

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL
})

api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem(ACCESS_TOKEN);
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

export default api
```

3. In .env file, add:

```
VITE_API_URL="http://localhost:8000"
```

## Task 4: Protected Routes

1. Create `components/ProtectedRoute.jsx`:

```jsx
import { Navigate } from "react-router-dom";
import { jwtDecode } from "jwt-decode";
import api from "../api";
import { REFRESH_TOKEN, ACCESS_TOKEN } from "../constants";
import { useState, useEffect } from "react";

function ProtectedRoute({ children }) {
  const [isAuthorized, setIsAuthorized] = useState(null);

  const refreshToken = async () => {
    const refreshToken = localStorage.getItem(REFRESH_TOKEN);
    try {
      const res = await api.post("/api/token/refresh", {
        refresh: refreshToken
      });
      if (res.status === 200) {
        localStorage.setItem(ACCESS_TOKEN, res.data.access);
        setIsAuthorized(true);
      } else {
        setIsAuthorized(false);
      }
    } catch (error) {
      console.log(error);
      setIsAuthorized(false);
    }
  };

  const auth = async () => {
    const token = localStorage.getItem(ACCESS_TOKEN);
    if (!token) {
      setIsAuthorized(false);
      return;
    }
    
    const decoded = jwtDecode(token);
    const tokenExpiration = decoded.exp;
    const now = Date.now() / 1000;
    
    if (tokenExpiration < now) {
      await refreshToken();
    } else {
      setIsAuthorized(true);
    }
  };

  useEffect(() => {
    auth();
  }, []);

  if (isAuthorized === null) {
    return <div>Loading...</div>;
  }
  
  return isAuthorized ? children : <Navigate to="/login" />;
}

export default ProtectedRoute;
```

## Task 5: Navigation & Pages

1. Create basic page components:

```jsx
// pages/Home.jsx
function Home() { 
  return <div>Home</div>
}

export default Home

// pages/Login.jsx
function Login() { 
  return <div>Login</div>
}

export default Login

// pages/NotFound.jsx
function NotFound() { 
  return (
    <>
      <h1>404 Not Found</h1>
      <p>The page you're looking for doesn't exist!</p>
    </>
  )
}

export default NotFound

// pages/Register.jsx
function Register() { 
  return <div>Register</div>
}

export default Register
```

2. Install router:

```bash
npm install react-router-dom
```

3. Set up App.jsx with routes:

```jsx
import React from "react";
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";

import Login from "./pages/Login";
import Register from "./pages/Register";
import Home from "./pages/Home";
import NotFound from "./pages/NotFound";
import ProtectedRoute from "./components/ProtectedRoute";

function Logout() {
  localStorage.clear();
  return <Navigate to="/login" />;
}

function RegisterAndLogin() {
  localStorage.clear();
  return <Register />;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route
          path="/"
          element={
            <ProtectedRoute>
              <Home />
            </ProtectedRoute>
          }
        />
        <Route path="/login" element={<Login />} />
        <Route path="/logout" element={<Logout />} />
        <Route path="/register" element={<RegisterAndLogin />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

## Task 6: Form Component

Create `components/Form.jsx`:

```jsx
import { useState } from "react";
import api from "../api";
import { useNavigate } from "react-router-dom";
import { ACCESS_TOKEN, REFRESH_TOKEN } from "../constants";
import "../styles/Form.css";


function Form({ route, method }) {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();

  const name = method === "login" ? "Login" : "Register";

  const handleSubmit = async (e) => {
    setLoading(true);
    e.preventDefault();

    try {
      const res = await api.post(route, { username, password });
      if (method === "login") {
        localStorage.setItem(ACCESS_TOKEN, res.data.access);
        localStorage.setItem(REFRESH_TOKEN, res.data.refresh);
        navigate("/");
      } else {
        navigate("/login");
      }
    } catch (error) {
      alert(error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="form-container">
      <h1>{name}</h1>
      <input
        className="form-input"
        type="text"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="Username"
      />
      <input
        className="form-input"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      {loading && <LoadingIndicator />}
      <button className="form-button" type="submit">
        {name}
      </button>
    </form>
  );
}

export default Form;
```

## Task 7: CSS Styling

1. Create `styles/Form.css`:

```css
.form-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 50px auto;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  max-width: 400px;
}

.form-input {
  width: 90%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.form-button {
  width: 95%;
  padding: 10px;
  margin: 20px 0;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.form-button:hover {
  background-color: #0056b3;
}
```

2. Create `styles/Note.css`:

```css
.note-container {
  padding: 10px;
  margin: 20px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.note-title {
  color: #333;
}

.note-content {
  color: #666;
}

.note-date {
  color: #999;
  font-size: 0.8rem;
}

.delete-button {
  background-color: #f44336; /* Red */
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.delete-button:hover {
  background-color: #d32f2f; /* Darker red */
}
```

3. Create `styles/Home.css`:

```css
/* Container for the whole page */
div {
  font-family: Arial, sans-serif;
}

/* Styles for the notes section */
.notes-section {
  margin-bottom: 2rem;
}

.notes-section h2 {
  color: #333;
  font-size: 24px;
}

/* Styles for individual notes */
.note {
  background-color: #f9f9f9;
  border-left: 5px solid #007bff;
  margin: 10px 0;
  padding: 10px 15px;
  border-radius: 5px;
}

/* Styles for the form section */
form {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-width: 500px;
  margin: auto;
}

form h2 {
  color: #333;
  font-size: 24px;
  margin-bottom: 20px;
}

form label {
  font-weight: bold;
  margin-top: 10px;
}

form input,
form textarea {
  width: 100%;
  padding: 8px;
  margin: 8px 0 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

form input[type="submit"] {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

form input[type="submit"]:hover {
  background-color: #0056b3;
}
```

4. Create `styles/LoadingIndicator.css`:

```css
.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.loader {
  border: 5px solid #f3f3f3; /* Light grey */
  border-top: 5px solid #3498db; /* Blue */
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
```

## Task 8: Component Implementation

1. Create `components/LoadingIndicator.jsx`:

```jsx
import "../styles/LoadingIndicator.css";

const LoadingIndicator = () => {
  return (
    <div className="loading-container">
      <div className="loader"></div>
    </div>
  );
};

export default LoadingIndicator;
```

2. Create `components/Note.jsx`:

```jsx
import React from "react";
import "../styles/Note.css";

function Note({ note, onDelete }) {
  const formattedDate = new Date(note.created_at).toLocaleDateString("en-US");

  return (
    <div className="note-container">
      <p className="note-title">{note.title}</p>
      <p className="note-content">{note.content}</p>
      <p className="note-date">{formattedDate}</p>
      <button className="delete-button" onClick={() => onDelete(note.id)}>
        Delete
      </button>
    </div>
  );
}

export default Note;
```

## Task 9: Home Page Implementation

Update `pages/Home.jsx`:

```jsx
import { useState, useEffect } from "react";
import api from "../api";
import Note from "../components/Note"
import "../styles/Home.css"

function Home() {
    const [notes, setNotes] = useState([]);
    const [content, setContent] = useState("");
    const [title, setTitle] = useState("");

    useEffect(() => {
        getNotes();
    }, []);

    const getNotes = () => {
        api
            .get("/api/notes/")
            .then((res) => res.data)
            .then((data) => {
                setNotes(data);
                console.log(data);
            })
            .catch((err) => alert(err));
    };

    const deleteNote = (id) => {
        api
            .delete(`/api/notes/delete/${id}/`)
            .then((res) => {
                if (res.status === 204) alert("Note deleted!");
                else alert("Failed to delete note.");
                getNotes();
            })
            .catch((error) => alert(error));
    };

    const createNote = (e) => {
        e.preventDefault();
        api
            .post("/api/notes/", { content, title })
            .then((res) => {
                if (res.status === 201) alert("Note created!");
                else alert("Failed to make note.");
                getNotes();
            })
            .catch((err) => alert(err));
    };

    return (
        <div>
            <div>
                <h2>Notes</h2>
                {notes.map((note) => (
                    <Note note={note} onDelete={deleteNote} key={note.id} />
                ))}
            </div>
            <h2>Create a Note</h2>
            <form onSubmit={createNote}>
                <label htmlFor="title">Title:</label>
                <br />
                <input
                    type="text"
                    id="title"
                    name="title"
                    required
                    onChange={(e) => setTitle(e.target.value)}
                    value={title}
                />
                <label htmlFor="content">Content:</label>
                <br />
                <textarea
                    id="content"
                    name="content"
                    required
                    value={content}
                    onChange={(e) => setContent(e.target.value)}
                ></textarea>
                <br />
                <input type="submit" value="Submit"></input>
            </form>
        </div>
    );
}

export default Home;
