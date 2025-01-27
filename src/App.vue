<template>
  <div class="container">
    <div class="message-board">
      <div v-if="!isLoggedIn" class="auth-container">
        <div class="auth-tabs">
          <button 
            @click="currentForm = 'login'"
            :class="{ active: currentForm === 'login' }"
          >
            Login
          </button>
          <button 
            @click="currentForm = 'register'"
            :class="{ active: currentForm === 'register' }"
          >
            Register
          </button>
        </div>
        <div class="auth-form">
        <form v-if="currentForm === 'login'" @submit.prevent="login">
          <input 
            v-model="loginData.email" 
            type="email" 
            placeholder="Email" 
            required 
          />
          <input 
            v-model="loginData.password" 
            type="password" 
            placeholder="Password" 
            required 
          />
          <button class="login-button" type="submit">Login</button>
        </form>

        <form v-else @submit.prevent="register">
          <input 
            v-model="registerData.username" 
            placeholder="Username" 
            required 
          />
          <input 
            v-model="registerData.email" 
            type="email" 
            placeholder="Email" 
            required 
          />
          <input 
            v-model="registerData.password" 
            type="password" 
            placeholder="Password" 
            required 
          />
          <button type="submit">Register</button>
        </form>
      </div>
      </div>

      <div v-else class="logged-in-container">
        <header>
          <h2>Welcome, {{ username }}</h2>
          <button @click="logout" class="logout-btn">Logout</button>
        </header>
        
        <form @submit.prevent="createPost" class="post-form">
          <input 
            v-model="newPost.title" 
            placeholder="Post Title" 
            required
            minlength="5"
            maxlength="256"
          />
          <textarea 
            v-model="newPost.text" 
            placeholder="Post Content" 
            required
            minlength="3"
            maxlength="2046"
          ></textarea>
          <button type="submit">Create Post</button>
        </form>

        <div class="posts-container">
          <div v-for="post in posts" :key="post._id" class="post">
            <h3>{{ post.title }}</h3>
            <p>By {{ post.user }}</p>
            <p>{{ post.text }}</p>
            <small>{{ formatDate(post.date) }}</small>
          </div>
        </div>
      </div>

      <div v-if="error" class="error">
        {{ error }}
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      currentForm: 'login',
      loginData: {
        email: '',
        password: ''
      },
      registerData: {
        username: '',
        email: '',
        password: ''
      },
      newPost: {
        title: '',
        text: ''
      },
      posts: [],
      error: null,
      isLoggedIn: false,
      username: '',
      token: ''
    }
  },
  methods: {
    async login() {
      try {
        const response = await axios.post('http://localhost:3000/auth/login', {
          email: this.loginData.email,
          password: this.loginData.password
        })
        
        this.token = response.data['auth-token']
        this.isLoggedIn = true

        const user = response.data['user']
        this.username = user.username

        this.error= ''
        
        await this.fetchPosts()
      } catch (err) {
        this.error = 'Login failed'
        console.error(err)
      }
    },
    async register() {
      try {
        await axios.post('http://localhost:3000/auth/register', {
          username: this.registerData.username,
          email: this.registerData.email,
          password: this.registerData.password
        })
        
        this.currentForm = 'login'
        this.error = 'Registration successful. Please log in.'
      } catch (err) {
        this.error = 'Registration failed'
        console.error(err)
      }
    },
    async fetchPosts() {
      try {
        const response = await axios.get('http://localhost:3000/posts', {
          headers: { 'auth-token': this.token }
        })
        this.posts = response.data
      } catch (err) {
        this.error = 'Failed to load posts'
        console.error(err)
      }
    },
    async createPost() {
      try {
        await axios.post('http://localhost:3000/posts', 
          {
            title: this.newPost.title,
            text: this.newPost.text,
          },
          { headers: { 'auth-token': this.token } }
        )
        
        await this.fetchPosts()
        this.newPost.title = ''
        this.newPost.text = ''
      } catch (err) {
        this.error = 'Failed to create post'
        console.error(err)
      }
    },
    formatDate(timestamp) {
      return new Date(timestamp).toLocaleString()
    },
    logout() {
      this.isLoggedIn = false
      this.token = ''
      this.username = ''
      this.posts = []
      this.currentForm = 'login'
    }
  }
}
</script>

<style scoped>

.container {
  width: 100vw;
  min-height: 100vh;
  padding: 30px;
  display: flex;
  justify-content: center;
  align-items: center;
  align-content: center;
  background-color: #689e9e; 
}

.message-board {
  width: 100%;
  max-width: 800px; 
  padding: 30px; 
  background-color: #ffffff; 
  border-radius: 8px; 
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); 
  display: flex; 
  flex-direction: column;
  gap: 20px; 
}

.post-form button {
  margin-left: auto; 
  display: block; 
  width: auto;; 
  max-width: 200px;
  padding: 10px 20px; 
  background-color: #28a745; 
  color: white; 
  border: none; 
  border-radius: 4px; 
  cursor: pointer; 
  font-size: 1rem; 
  transition: background-color 0.3s ease;
}

.post-form button:hover {
  background-color: #218838;
}

.auth-container, .logged-in-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.auth-container button {
  margin: auto;
  padding: 12px 20px;
  max-width: 200px;
  border: none;
  background-color: #e7e7e7;
  color: #333;
  transition: background-color 0.3s ease;
}

.auth-container button:hover {
  background-color: #aaaaaa; 
}

.auth-tabs {
  display: flex;
  margin-bottom: 20px;
}

.auth-tabs button {
  flex-grow: 1;
  max-width: 600px;
  padding: 12px;
  border: none;
  background-color: #e7e7e7;
  color: #333;
  transition: background-color 0.3s ease;
}

.auth-tabs button:hover {
  background-color: #aaaaaa; 
}

.auth-tabs button.active {
  background-color: #007bff;
  color: white;
}

.auth-form input {
  width: 350px;
  margin: auto;
}

form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}


input, textarea {
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  resize: none;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.logout-btn {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
}

.posts-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.post {
  background-color: #f9f9f9;
  border: 1px solid #e7e7e7;
  border-radius: 6px;
  padding: 20px;
}

.post h3 {
  margin-top: 0;
  color: #333;
}

.post small {
  color: #666;
  font-size: 0.8em;
}



.error {
  color: red;
  text-align: center;
  margin-top: 15px;
}

/* @media (max-width: 768px) {
  .message-board {
    max-width: 100%;
    margin: 0;
    padding: 20px;
  }
}

@media (min-width: 1024px) {
  .message-board {
    max-width: 700px; 
    margin: auto;
  }
} */

</style>