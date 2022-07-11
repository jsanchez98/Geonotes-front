<template>
  <!--<HelloWorld msg="Welcome to Your Vue.js App" /> -->
  <div id="background">
    <h1>MICROBLOGS</h1>
    <button @click="logout" v-if="loggedin">log out</button>
    <div class="maindiv max-w-sm rounded overflow-hidden shadow-lg">
      <div id="content" v-if="loggedin">
        <BlogPost
          v-for="post in blogPosts"
          :key="post.ID"
          :text="post.text"
          :index="blogPosts.indexOf(post)"
          :id="post.ID"
          @delete="deletePost"
        />
        <div id="blogform">
          <form class="bg-white rounded px-8 pt-6 pb-8 mb-4">
            <input
              v-model="text"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
              type="text"
              @keyup.enter="addPost"
            />
          </form>
          <button
            @click="addPost"
            class="submitbutton bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
            type="button"
          >
            submit
          </button>
        </div>
      </div>
      <div id="content" v-else>
        <LoginPage
          :csrfToken="csrfToken"
          @login="loginLoad"
          @register="loginLoad"
        />
      </div>
    </div>
    {{ statusMessage }}
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue';
import BlogPost from "./components/BlogPost.vue";
import LoginPage from "./components/LoginPage.vue";
import axios from "axios";
const path = "http://localhost:5000/";

export default {
  name: "App",
  components: {
    //HelloWorld,
    BlogPost,
    LoginPage,
  },
  data() {
    return {
      blogPosts: [],
      list: ["one", "two"],
      msg: "ms",
      text: "",
      loggedin: false,
      csrfToken: "",
      statusMessage: "",
      axiosConfig: {
          headers: {
            "X-CSRFToken": this.csrfToken,
          },
          withCredentials: true,
        }
    };
  },
  created() {
    this.fetchCSRFToken();
    if (this.loggedin) {
      this.loadPosts();
    }
  },
  methods: {
    deletePost(index, id) {
      const path = "http://localhost:5000/deletepost";
      this.blogPosts.splice(index, 1);
      axios
        .post(path, {
          postID: id,
        },
        {
          headers: {
            "X-CSRFToken": this.csrfToken,
          },
          withCredentials: true,
        })
        .then(() => {
          
        })
        .catch(() => {
          this.statusMessage = "Delete Failed"
          this.loadPosts()
        });
    },
    addPost(event) {
      event.preventDefault();
      if (this.text != "") {
        const path = "http://localhost:5000/addpost";
        axios
          .post(path, {
            text: this.text,
            id: 1,
          },
          {
          headers: {
            "X-CSRFToken": this.csrfToken,
          },
          withCredentials: true,
        })
          .then((res) => {
            this.blogPosts.push({
              text: res.data.text,
              ID: res.data.ID,
            });
            this.text = "";
          })
          .catch((error) => {
            alert(error);
          });
      } else {
        return;
      }
    },
    loadPosts() {
      const path = "http://localhost:5000/posts";
      axios
        .get(path, {
          headers: {
            "X-CSRFToken": this.csrfToken,
          },
          withCredentials: true,
        })
        .then((res) => {
          this.blogPosts = res.data;
          this.loggedin = true;
        })
        .catch((error) => {
          alert("here " + error.message);
        });
    },
    loginLoad() {
      this.loadPosts();
    },
    logout() {
      const path = "http://localhost:5000/logout";
      axios
        .get(path, {
          headers: {
            "X-CSRFToken": this.csrfToken,
          },
          withCredentials: true,
        })
        .then(() => {
          this.loggedin = false;
        })
        .catch((error) => {
          alert("logout failed " + error.message);
        });
    },
    fetchCSRFToken() {
      axios
        .get(path + "/api/csrf", this.axiosConfig)
        .then((res) => {
          this.csrfToken = res.headers["x-csrftoken"];
        })
        .catch((err) => {
          alert(err.message);
        });
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#content {
  margin: 10px 5px 5px 0px;
}

#blogform {
  display: flex;
  margin-top: 10px;
  align-items: center;
}

.submitbutton {
  margin-left: 20px;
  width: 100px;
  height: 50px;
}

.maindiv {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  margin: auto;
  margin-top: 5%;
  background-color: white;
}

#background {
  position: fixed;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  background-color: #c8ecfb;
}
</style>
