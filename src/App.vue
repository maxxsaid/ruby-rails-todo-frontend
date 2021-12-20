<template>
  <!-- <div id="app">
    <div id="nav">
      <Header />
    </div>
    <router-view />
    <Footer />
  </div> -->
  <div id="app">
    <h1 id="title">max<span>todos</span></h1>
    <div id="nologin" v-if="!loggedin">
      <div id="loginform">
        <form v-on:submit.prevent="handleLogin">
          <fieldset>
            <legend>LOGIN</legend>
            <input type="text" placeholder="username" v-model="loginUsername" />
            <input
              type="password"
              placeholder="password"
              v-model="loginPassword"
            />
            <input type="submit" value="login" />
          </fieldset>
        </form>
        <form v-on:submit.prevent="handleSignup">
          <fieldset>
            <legend>Create User</legend>
            <input
              type="text"
              placeholder="username"
              v-model="createUsername"
            />
            <input
              type="password"
              placeholder="password"
              v-model="createPassword"
            />
            <input type="submit" value="create user" />
          </fieldset>
        </form>
      </div>
    </div>
    <div id="login" v-if="loggedin">
      <div id="forms">
        <article>
          <h1>New ToDo</h1>
          <input type="text" v-model="newTodo" />
          <button v-on:click.prevent="createTodo">Create ToDo</button>
        </article>
        <article>
          <h1>Update ToDo</h1>
          <input type="text" v-model="updateTodo" />
          <button v-on:click.prevent="editTodo" v-bind:id="editID">
            Update ToDo
          </button>
        </article>
      </div>
      <div id="todos">
        <div class="todo" v-for="todo of todos" :key="todo.id">
          <h1>{{ todo.message }}</h1>
          <button v-bind:id="todo.id" v-on:click="deleteTodo">delete</button>
          <button v-bind:id="todo.id" v-on:click="editSelect">update</button>
        </div>
      </div>
      <button v-on:click="handleLogout" id="logout">logout</button>
    </div>
  </div>
</template>

<script>
// import Header from './components/Header'
// import Footer from './components/Footer'

export default {
  name: "App",
  // components: {
  //   Header,
  //   Footer
  // }
  data: function() {
    return {
      loggedin: false,
      JWT: "",
      loginUsername: "",
      loginPassword: "",
      createUsername: "",
      createPassword: "",
      devURL: "http://localhost:3000",
      prodURL: "https://rubyonrails-todo-backend.herokuapp.com",
      user: null,
      token: null,
      todos: [],
      newTodo: "",
      updateTodo: "",
      editID: 0,
    };
  },
  methods: {
    handleLogin: function() {
      const URL = this.prodURL ? this.prodURL : this.devURL;
      const user = {
        username: this.loginUsername,
        password: this.loginPassword,
      };
      fetch(`${URL}/login`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(user),
      })
        .then((response) => response.json())
        .then((data) => {
          if (data.error) {
            alert("error logging in");
          } else {
            this.user = data.user;
            this.token = data.token;
            this.loggedin = true;
            this.getTodos();
            this.loginPassword = "";
            this.loginUsername = "";
            window.sessionStorage.setItem("login", JSON.stringify(data));
          }
        });
    },
    handleLogout: function() {
      this.loggedin = false;
      this.user = null;
      this.token = null;
      window.sessionStorage.removeItem("login");
    },
    handleSignup: function() {
      const URL = this.prodURL ? this.prodURL : this.devURL;
      const user = {
        username: this.createUsername,
        password: this.createPassword,
      };
      fetch(`${URL}/users`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: user,
      })
        .then((response) => response.json())
        .then((data) => {
          if (data.error) {
            alert("sign up unsuccessul");
          } else {
            alert("sign up successful");
            this.createUsername = "";
            this.createPassword = "";
          }
        });
    },
    getTodos: function() {
      const URL = this.prodURL ? this.prodURL : this.devURL;
      fetch(`${URL}/todos`, {
        method: "get",
        headers: {
          Authorization: `bearer ${this.token}`,
        },
      })
        .then((response) => response.json())
        .then((data) => {
          this.todos = data;
        });
    },
    createTodo: function() {
      const URL = this.prodURL ? this.prodURL : this.devURL;
      const todo = { message: this.newTodo };

      fetch(`${URL}/todos`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `bearer ${this.token}`,
        },
        body: JSON.stringify(todo),
      }).then((response) => {
        this.newTodo = "";
        this.getTodos();
      });
    },
    deleteTodo: function(event) {
      const URL = this.prodURL ? this.prodURL : this.devURL;
      const ID = event.target.id;

      fetch(`${URL}/todos/${ID}`, {
        method: "DELETE",
        headers: {
          Authorization: `bearer ${this.token}`,
        },
      }).then((response) => {
        this.getTodos();
      });
    },
    editTodo: function(event) {
      const URL = this.prodURL ? this.prodURL : this.devURL;
      const ID = event.target.id;
      const updated = { message: this.updateTodo };

      fetch(`${URL}/todos/${ID}`, {
        method: "PUT",
        headers: {
          Authorization: `bearer ${this.token}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify(updated),
      }).then((response) => {
        this.getTodos();
      });
    },
    editSelect: function(event) {
      this.editID = event.target.id;
      const theTodo = this.todos.find((todo) => {
        return todo.id == this.editID;
      });
      this.updateTodo = theTodo.message;
    },
  },
  created: function() {
    const getLogin = JSON.parse(window.sessionStorage.getItem("login"));
    if (getLogin) {
      this.user = getLogin.user;
      this.token = getLogin.token;
      this.loggedin = true;
      this.getTodos();
    }
  },
};
</script>

<style>
#title {
  text-align: center;
  margin: 50px auto;
  font-size: 4em;
}
#title span {
  color: aqua;
}

#loginform {
  display: flex;
  width: 60%;
  margin: 10px auto;
}
#forms {
  width: 60%;
  margin: 10px auto;
  display: flex;
  justify-content: space-between;
}
#todos {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  width: 80%;
  margin: 10px auto;
}
#todos div {
  margin: 15px;
  max-width: 300px;
  padding: 10px;
  border: 2px solid navy;
}
#logout {
  text-align: center;
  display: block;
  margin: 10px auto;
}
</style>
