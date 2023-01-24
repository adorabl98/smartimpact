<template>
  <nav class="header">
    <div class="header-left">
      <img src="../assets/github-mark.svg" alt="Github logo" class="logo" />
      <h1 class="home-title">GITHUB</h1>
    </div>
    <p class="api-title">Smartimpact demo</p>
    <SearchBar class="search-bar" @updateData="update" />
  </nav>
  <div class="main-info">
    <p v-if="!dataLoaded" class="intro-message">{{ message }}</p>
    <template v-if="dataLoaded">
      <div class="info-wrapper">
        <div v-if="user != null" class="user-wrapper">
          <img :src="user.avatar_url" alt="User Avatar" class="avatar" />
          <div class="user-wrapper__info">
            <div>
              <p class="info important" v-if="user.login != null">
                {{ user.name }} ({{ user.login }})
              </p>
              <p class="info" v-if="user.location != null">
                Location: {{ user.location }}
              </p>
              <p class="info" v-if="user.bio != null">{{ user.bio }}</p>
              <p class="info" v-if="user.followers != null">
                Followers: {{ user.followers }}
              </p>
              <p class="info" v-if="user.following != null">
                Following: {{ user.following }}
              </p>
            </div>
            <a
              v-if="user.html_url != null"
              :href="user.html_url"
              target="_blank"
              class="user_url"
              >I need more info</a
            >
          </div>
        </div>
        <div class="list-wrapper">
          <p class="intro-message" v-if="gists.length == 0">
            This user has no gists...
          </p>
          <div v-else>
            <p class="intro-message">Gists</p>
            <div class="gists-wrapper">
              <div
                class="gist-card"
                v-for="(gist, gist_index) in gists"
                :key="gist.id"
              >
                <div class="info-title-wrapper">
                  <img
                    :src="gist.owner.avatar_url"
                    alt="Gist user Avatar"
                    class="gist_user_avatar"
                  />
                  <div class="gist-card__info">
                    <a
                      class="gist-card__info_title"
                      :title="
                        gist.description != ''
                          ? gist.description
                          : 'No description'
                      "
                      target="blank"
                      :href="gist.html_url"
                    >
                      {{
                        gist.description != ""
                          ? gist.description
                          : "No description"
                      }}
                    </a>
                    <p class="gist-card__info__created_at">
                      {{
                        gist.created_at != null
                          ? new Date(gist.created_at).toDateString()
                          : "-"
                      }}
                    </p>
                  </div>
                </div>
                <div
                  v-if="Object.keys(gist.files).length > 0"
                  class="gist-card__files_list"
                >
                  <div
                    v-for="(item, file_index) in gist.files"
                    :key="item.filename"
                    :id="gist_index + '-' + item.filename"
                  >
                    <div :class="['gist-file']">
                      <div
                        class="file-dropdown"
                        @click="
                          loadGist(
                            item.raw_url,
                            gist.forks_url,
                            gist_index,
                            file_index
                          )
                        "
                      >
                        <p class="filename">{{ item.filename }}</p>
                      </div>
                      <pre class="code"><code>Loading...</code></pre>
                      <div class="forks">
                        Forks:
                        <div class="forks_wrapper"></div>
                      </div>
                    </div>
                  </div>
                </div>
                <div
                  v-if="Object.keys(gist.files).length > 0"
                  class="gist-card__files"
                >
                  <div v-for="item in gist.files" :key="item.filename">
                    <div
                      v-if="item.language"
                      :class="['gist-card__files_language']"
                    >
                      {{ item.language }}
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import SearchBar from "../components/SearchBarForm.vue";
import axios from "axios";
// import SshPre from "simple-syntax-highlighter";
import "simple-syntax-highlighter/dist/sshpre.css";
import "highlight.js/styles/github.css";
export default {
  name: "home-page",

  components: {
    SearchBar,
    // SshPre,
  },

  data() {
    return {
      dataLoaded: false,
      message: "Search for an user and see informations about him!",
      user: null,
      gists: [],
    };
  },

  mounted() {
    this.init();
  },

  methods: {
    init() {
      //
    },

    update(response) {
      console.log(response.gists);
      if (response.status == true) {
        this.dataLoaded = true;
        this.user = response.user;
        this.gists = response.gists;
      } else {
        this.dataLoaded = false;
        this.user = null;
        this.gists = [];
        this.message = response.status;
      }
    },

    async loadGist(file_url, forks_url, gist_index, file_key) {
      document
        .getElementById(gist_index + "-" + file_key)
        .classList.toggle("expanded");
      if (
        document
          .getElementById(gist_index + "-" + file_key)
          .classList.contains("expanded")
      ) {
        try {
          const request = await axios.get(file_url);
          if (typeof request !== "undefined") {
            //
            var hljs = require("highlight.js");
            var html = hljs.highlightAuto(request.data).value;

            document.getElementById(
              gist_index + "-" + file_key
            ).children[0].children[1].innerHTML = html;
          }
        } catch (e) {
          document.getElementById(
            gist_index + "-" + file_key
          ).children[0].children[1].innerText = "Something went wrong...";
        }
      }

      try {
        const forks_request = await axios.get(forks_url);
        if (typeof forks_request !== "undefined") {
          let forks = forks_request.data;
          if (forks.length > 0) {
            for (let i = 0; i < forks.length; i++) {
              var wrapper = document.createElement("div");

              var image = document.createElement("img");
              image.src = forks[i].owner.avatar_url;

              var name = document.createElement("p");
              name.innerText = forks[i].owner.login;
              wrapper.appendChild(image);
              wrapper.appendChild(name);

              document
                .getElementById(gist_index + "-" + file_key)
                .children[0].children[2].children[0].appendChild(wrapper);
            }
          } else {
            document.getElementById(
              gist_index + "-" + file_key
            ).children[0].children[2].innerText = "No forks found";
          }
        }
      } catch (e) {
        //
      }
    },

    getLanguage(string) {
      if (string == null) return "-";

      if (string == "JavaScript") return "js";
      return string.toLowerCase();
    },
  },
};
</script>

<style lang="scss">
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  border-inline: 1px solid black;
  border-bottom: 1px solid black;
  border-bottom-left-radius: 0.625rem;
  border-bottom-right-radius: 0.625rem;

  margin-inline: auto;

  max-width: 1920px;

  @media screen and (max-width: 40rem) {
    flex-direction: column;
    align-items: flex-start;
    justify-content: start;
  }

  .header-left {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 0.5rem;
    flex: 1;
    @media screen and (max-width: 40rem) {
      width: 100%;
      margin-right: 0;
      margin-bottom: 1rem;
    }

    .logo {
      width: 3rem;
      height: 3rem;
      margin-right: 1rem;
    }
  }

  .api-title {
    flex: 1;
    font-size: 1.2rem;
    font-weight: 600;
    text-transform: uppercase;
    text-align: center;
    margin-left: auto;
    margin-right: auto;
    @media screen and (max-width: 40rem) {
      margin-bottom: 1rem;
    }
  }

  .search-bar {
    flex: 2;
  }
}

.main-info {
  margin-inline: auto;
  padding-block: 1rem;
  max-width: 1920px;

  .intro-message {
    font-size: 1.5rem;
    text-align: center;
    padding-block: 1rem;
  }
}

.info-wrapper {
  width: 100%;
  .user-wrapper {
    width: 100%;
    padding-bottom: 1rem;
    margin-bottom: 1rem;
    border-bottom: 1px solid #131313;
    display: flex;
    align-items: center;

    @media screen and (max-width: 40rem) {
      flex-direction: column;
      align-items: center;
    }

    .avatar {
      flex: 1;
      max-width: 300px;
      border-radius: 10%;
      width: 100%;
      object-fit: cover;
      margin-right: 1rem;
      @media screen and (max-width: 40rem) {
        max-width: 200px;
        margin-right: 0rem;
        margin-bottom: 1rem;
      }
    }

    .user-wrapper__info {
      flex: 2;

      @media screen and (max-width: 40rem) {
        width: 100%;
      }
      .info {
        font-size: 1rem;
        margin-bottom: 0.5rem;
        &.important {
          font-size: 1.2rem;
          font-weight: bold;
        }
      }

      .user_url {
        display: block;
        box-sizing: border-box;
        width: fit-content;
        text-decoration: none;
        color: #131313;
        font-size: 1rem;
        padding: 0.5rem 1rem;
        border-radius: 0.625rem;
        background-color: #131313;
        border: 1px solid #131313;
        color: white;
        &:hover {
          cursor: pointer;
          background-color: white;
          color: #131313;
        }

        @media screen and (max-width: 40rem) {
          margin-inline: auto;
          width: 100%;
          text-align: center;
        }
      }
    }
  }

  .list-wrapper {
    width: 100%;
    margin-bottom: 1rem;
  }

  .gists-wrapper {
    margin-block: -0.5rem;
    display: flex;
    flex-direction: column;
  }
  .gist-card {
    padding: 1rem;
    border-radius: 0.625rem;
    background: rgb(197, 197, 197, 0.5);
    margin-bottom: 0.5rem;
    margin-inline: 0.5rem;
    display: flex;
    flex-direction: column;
    height: 100%;

    .gist_user_avatar {
      width: 2.5rem;
      height: 2.5rem;
      border-radius: 100%;
      margin-right: 1rem;
      border: 2px solid gray;
    }

    .info-title-wrapper {
      display: flex;
      flex-direction: row;
      align-items: center;
      margin-bottom: 1rem;
      .gist-card__info {
        .gist-card__info_title {
          font-weight: bold;
          margin-block: 0.5rem;
          overflow: hidden;
          text-overflow: ellipsis;
          display: -webkit-box;
          -webkit-line-clamp: 1;
          line-clamp: 1;
          -webkit-box-orient: vertical;
          cursor: pointer;
          text-decoration: none;
          color: #0000aa;

          &:hover {
            text-decoration: underline;
          }
        }

        .gist-card__info__created_at {
          font-size: 0.8rem;
        }
      }
    }

    .gist-card__files_list {
      .file-dropdown {
        padding-bottom: 0.5rem;
        margin-bottom: 1rem;
        border-bottom: 1px solid #131313;
        .filename {
          font-weight: bold;
          overflow: hidden;
          text-overflow: ellipsis;
          display: -webkit-box;
          -webkit-line-clamp: 1;
          line-clamp: 1;
          -webkit-box-orient: vertical;
          cursor: pointer;
          text-decoration: none;
          color: #0000aa;
          font-size: 0.9rem;
        }
      }
    }

    .gist-file {
      .code,
      .forks {
        display: none;
      }
    }

    .expanded {
      .code,
      .forks {
        padding: 1rem;
        width: 100%;
        display: block;
        margin-bottom: 1rem;
        box-sizing: border-box;
      }

      .code {
        color: white;
        background: #1e2029;
        border-radius: 0.625rem;
      }

      .forks_wrapper {
        margin-top: 0.5rem;
        display: flex;
        flex-wrap: wrap;
        div {
          margin-right: 0.5rem;
          display: flex;
          align-items: center;
          img {
            width: 2rem;
            height: 2rem;
            margin-right: 0.5rem;
            border-radius: 100%;
            border: 3px solid gray;
          }
        }
      }
    }

    .gist-card__files {
      margin-bottom: 1rem;
      display: flex;
      flex-wrap: wrap;

      .gist-card__files_language {
        margin-right: 0.5rem;
        margin-bottom: 0.5rem;
        border-radius: 0.625rem;
        padding: 0.3rem 0.7rem;
        width: fit-content;
        font-size: 0.9rem;

        &:nth-child(5n + 1) {
          background-color: #131313;
          color: white;
        }
        &:nth-child(5n + 2) {
          background-color: #202020;
          color: white;
        }
        &:nth-child(5n + 3) {
          background-color: #292929;
          color: white;
        }
        &:nth-child(5n + 4) {
          background-color: #777;
          color: white;
        }
        &:nth-child(5n + 5) {
          background-color: #aaa;
          color: black;
        }
      }
    }
  }
}
</style>
