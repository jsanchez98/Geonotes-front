<template>
  <div id="login">
    <h3>
      {{ errorMessage }}
    </h3>
    <button @click="toggleLogin">
      <div v-if="!loginFlag" class="hover:text-blue-600">Already got an account? Log in</div>
      <a v-else class="hover:text-blue-600">Don't have an account? Register for one now</a>
    </button>
    <form>
      <input
        v-model="username"
        placeholder="username"
        class="mx-1 mt-3 shadow appearance-none border rounded py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
      />
      <input
        v-model="password"
        placeholder="password"
        class="mx-1 my-3 shadow appearance-none border rounded py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
      />
    </form>
    <button @click="login" v-if="loginFlag" class="bg-white hover:bg-gray-100 font-semibold py-2 px-4 border rounded border-gray-400 shadow ">Log In</button>
    <button @click="register" v-else class="bg-white hover:bg-gray-100 font-semibold py-2 px-4 border rounded border-gray-400 shadow ">Register</button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      username: "",
      password: "",
      loginFlag: true,
      errorMessage: ""
    };
  },
  props: {
    csrfToken: String
  },
  methods: {
    login() {
      const path = "http://localhost:5000/login";
      const config = {
        headers: {
          "Content-Type": "application/json",
          "X-CSRFToken": this.csrfToken,
        },
        withCredentials: "include"
      }
      axios
        .post(
          path,
          {
            username: this.username,
            password: this.password,
          },
          config
        )
        .then(() => {
          this.$emit("login");
        })
        .catch((err) => {
          if (err.response.data.message == "Invalid username or password") {
            this.setErrorMessage("Invalid username or password");
          }
        });
    },
    register() {
      const path = "http://localhost:5000/register";
      axios
        .post(path, {
          username: this.username,
          password: this.password,
        })
        .then(() => {
          this.$emit("register");
          this.username = "";
          this.password = "";
        })
        .catch((err) => {
          if (err.response.data.message == "Username taken") {
            this.setErrorMessage("Username taken");
          }
        });
    },
    setErrorMessage(message) {
      this.errorMessage = message;
      setTimeout(() => {
        this.errorMessage = "";
      }, 3000);
    },
    toggleLogin() {
      this.loginFlag = !this.loginFlag;
    },
  },
};
</script>

<style scoped>
#login {
  width: 100%;
  height: 100%;
}
</style>
