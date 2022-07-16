<template>
  <!--<HelloWorld msg="Welcome to Your Vue.js App" /> -->
  <div id="background">
    <div class="header">
      <button style="color:white" class="button" @click="logout" v-if="loggedin">log out</button>
      <h1 style="color:white">GEONOTES</h1>
    </div>
    <div id="layout">
      <div id="maindiv" :style="{top: maindivTop, left: maindivLeft}" class="max-w-sm rounded overflow-hidden shadow-lg">
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
        <h1 v-if="loggedin && pointToSet"> {{ JSON.stringify(pointToSet.geometry.coordinates) }} </h1>
      </div>
      <div v-if="loggedin" class="mapdiv rounded overflow-hidden shadow-lg">
        <NoteMap @setPoint="setPoint" :posts="blogPosts"/>
      </div>
    </div>
    {{ statusMessage }}
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue';
import BlogPost from "./components/BlogPost.vue";
import LoginPage from "./components/LoginPage.vue";
import NoteMap from "./components/NoteMap.vue";
import axios from "axios";
const path = "http://localhost:5000/";

export default {
  name: "App",
  components: {
    //HelloWorld,
    BlogPost,
    LoginPage,
    NoteMap,
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
      },
      pointToSet: null,
      maindivTop: "30vh",
      maindivLeft: "36.5vw"
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
        .post(
          path,
          {
            postID: id,
          },
          {
            headers: {
              "X-CSRFToken": this.csrfToken,
            },
            withCredentials: true,
          }
        )
        .then(() => {})
        .catch(() => {
          this.statusMessage = "Delete Failed";
          this.loadPosts();
        });
    },
    addPost(event) {
      event.preventDefault();
      if (this.text != "" && (this.pointToSet != null)) {
        const path = "http://localhost:5000/addpost";
        axios
          .post(
            path,
            {
              text: this.text,
              coordinates: JSON.stringify(this.pointToSet.geometry.coordinates)
            },
            {
              headers: {
                "X-CSRFToken": this.csrfToken,
              },
              withCredentials: true,
            }
          )
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
        alert("failed")
        this.timeoutStatusMessage("Please choose a location for this post")
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
          this.maindivTop = "0px";
          this.maindivLeft= "0px"
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
          this.maindivTop= "30vh"
          this.maindivLeft= "36.5vw"
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
    timeoutStatusMessage(message){
      this.statusMessage = message
      setTimeout(() => {
        this.statusMessage = "";
      }, 3000);
    },
    setPoint(point){
      this.pointToSet = point
    }
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

.header {
  display: flex;
  justify-content: space-around;
}

#layout {
  position: relative;
  width: 100%;
  height: 95%;
  left: 0;
  top: 0;
  margin-top: 5px;
}

#button {
  left: 10%;
  top: 10%;
}

.mapdiv {
  position: absolute;
  width: 100%;
  height: 100%;
  top:0;
  left:0;
  pointer-events: all;
}

#content {
  margin: 10px 5px 5px 5px;
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

#maindiv {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  margin-bottom: auto;
  margin-left: 20px;
  background-color: #ede7da;
  position: absolute;
  z-index: 1;
  top: 30vh;
  left: 36.5vw;
  transition: transform 1s;
}
/*
@keyframes change_colour {
  0%, 100% {
    background-color: #105666;
  }
  100% {
    background-color: #ed7a72;
  }
}*/

#background {
  position: fixed;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  background-color: #ed7a72;
}
</style>
