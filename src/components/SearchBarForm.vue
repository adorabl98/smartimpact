<template>
  <form
    class="search-form"
    ref="search-form"
    @submit.prevent="searchUsername()"
    autocomplete="off"
  >
    <input
      autocomplete="false"
      name="hidden"
      type="text"
      style="display: none"
    />
    <div class="form-item">
      <input
        v-model="username"
        type="text"
        name="username"
        placeholder="Username"
      />
      <p class="error">{{ errors.username }}</p>
    </div>
    <button type="submit">Search</button>
  </form>
</template>

<script>
import axios from "axios";

export default {
  metaInfo: {
    title: "",
  },
  name: "SearchBarForm",
  data: function () {
    return {
      username: "",
      errors: {
        username: "",
      },
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
    validateForm() {
      this.errors.username = "";
      if (this.username.length == 0) {
        this.errors.username = "Username is required.";
        return false;
      }
      if (this.username[0] == "-") {
        this.errors.username = "Username cannot start with '-'";
        return false;
      }
      if (this.username.length > 39) {
        this.errors.username = "Username must be shorter than 39 characters.";
        return false;
      }
      return true;
    },

    async searchUsername() {
      if (!this.validateForm()) {
        return -1;
      }
      this.$emit("updateData", {
        status: "Loading...",
        user: null,
        gists: null,
      });
      try {
        const userRequest = await axios.get(
          "https://api.github.com/users/" + this.username + "",
          {},
          {
            headers: {
              Accept: "application/vnd.github+json",
              Authorization: "Bearer " + process.env.VUE_APP_GITHUB_TOKEN,
              "X-GitHub-Api-Version": "2022-11-28",
            },
          }
        );
        if (typeof userRequest !== "undefined") {
          this.user = userRequest.data;
          const gistsRequest = await axios.get(
            "https://api.github.com/users/" + this.username + "/gists",
            {},
            {
              headers: {
                Accept: "application/vnd.github+json",
                Authorization: "Bearer " + process.env.VUE_APP_GITHUB_TOKEN,
                "X-GitHub-Api-Version": "2022-11-28",
              },
            }
          );
          if (typeof gistsRequest !== "undefined") {
            this.gists = gistsRequest.data;
          }
          this.$emit("updateData", {
            status: true,
            user: this.user,
            gists: this.gists,
          });
        }
      } catch (e) {
        if (e.response != undefined && e.response.status == 404) {
          this.$emit("updateData", {
            status: "Username not found! Please check for spelling errors.",
            user: null,
            gists: null,
          });
        } else
          this.$emit("updateData", {
            status: "Something went wrong. Please try again later!",
            user: null,
            gists: null,
          });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.search-form {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding-block: 1rem;
  position: relative;

  button {
    padding: 0.5rem;
    border-radius: 0.625rem;
    font-size: 1rem;
    background-color: #131313;
    border: 1px solid #131313;
    color: white;
    &:hover {
      cursor: pointer;
      background-color: white;
      color: #131313;
    }
  }
}
.form-item {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: start;
  margin-left: auto;
  margin-right: 1rem;
  padding-right: 1rem;
  width: 100%;
  max-width: 400px;

  @media screen and (max-width: 40rem) {
    margin-left: unset;
  }

  label {
    margin-bottom: 0.5rem;
    font-size: 1rem;
    font-weight: 600;
  }

  input {
    border: 1px solid #131313;
    padding: 0.5rem;
    border-radius: 0.625rem;
    font-size: 1rem;
    width: 100%;
  }

  .error {
    position: absolute;
    bottom: 0;
    font-size: 0.8rem;
    color: red;
  }
}
</style>
