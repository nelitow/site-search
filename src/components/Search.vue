<template>
  <form class="search-form">
    <input
      type="search"
      class="search-input"
      v-model="searchString"
      placeholder="Enter search term..."
      @focus="showResults = 1"
      @blur="showResults = 0"
    />
    <button class="search-button">
      <svg class="submit-button">
        <use
          xmlns:xlink="http://www.w3.org/1999/xlink"
          xlink:href="#search"
        ></use>
      </svg>
    </button>
  </form>

  <svg xmlns="http://www.w3.org/2000/svg" width="0" height="0" display="none">
    <symbol id="search" viewBox="0 0 32 32">
      <path
        d="M 19.5 3 C 14.26514 3 10 7.2651394 10 12.5 C 10 14.749977 10.810825 16.807458 12.125 18.4375 L 3.28125 27.28125 L 4.71875 28.71875 L 13.5625 19.875 C 15.192542 21.189175 17.250023 22 19.5 22 C 24.73486 22 29 17.73486 29 12.5 C 29 7.2651394 24.73486 3 19.5 3 z M 19.5 5 C 23.65398 5 27 8.3460198 27 12.5 C 27 16.65398 23.65398 20 19.5 20 C 15.34602 20 12 16.65398 12 12.5 C 12 8.3460198 15.34602 5 19.5 5 z"
      />
    </symbol>
  </svg>
  <transition name="fade">
    <div class="results" v-if="showResults && searchString.length > 3">
      <div v-for="item in urlSet" :key="item.url" v-show="pageMatch(item.data)">
        <div>
          <a :href="item.url" target="_blank" rel="noopener noreferrer">
            {{ item.title }}
          </a>
          <hr />
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import axios from "axios";
var parseString = require("xml2js").parseString;

export default {
  data() {
    return {
      showResults: 0,
      searchString: "",
      urlSet: []
    };
  },
  methods: {
    pageMatch(data) {
      try {
        return data.includes(this.searchString);
      } catch (error) {
        console.log("No page data yet.");
        return false;
      }
    }
  },
  created(data) {
    axios
      .get("./sitemap.xml", {})
      .then(response => {
        parseString(response.data, (err, result) => {
          result.urlset.url.forEach((el, i) => {
            this.urlSet.push({ url: el.loc[0] });
          });
        });
      })
      .then(() => {
        this.urlSet.forEach((el, i) => {
          axios.get(el.url).then(response => {
            let DOMresponse = new DOMParser().parseFromString(
              response.data.content,
              "text/html"
            );
            this.urlSet[i].data =
              DOMresponse.documentElement.textContent.trim() + " ";
            this.urlSet[i].title = DOMresponse.getElementsByTagName(
              "h1"
            )[0].textContent;
            console.log(DOMresponse);
          });
        });
      });
  }
};
</script>

<style lang="scss" scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.results {
  width: 100%;
  padding: 16px;
  box-shadow: 0 3px 4px rgba(0, 0, 0, 0.15);
  font-family: sans-serif;
  border-radius: 20px;
  color: #5a6674;
  box-sizing: border-box;
  border: 1px solid rgba(0, 0, 0, 0.05);
  a {
    color: #5a6674cc;
    transition: color 0.2s ease;
    text-decoration: none;
    &:hover {
      color: #5a6674;
    }
  }
  hr {
    border: 0;
    height: 1px;
    background: rgba(0, 0, 0, 0.05);
  }
}
.search-form {
  position: relative;
  width: 350px;
  height: 40px;
  border-radius: 40px;
  box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
  background: #fff;
  transition: all 0.3s ease;
  margin-bottom: 20px;

  &.focus {
    box-shadow: 0 3px 4px rgba(0, 0, 0, 0.15);
  }

  .search-input {
    position: absolute;
    top: 10px;
    left: 38px;
    font-size: 14px;
    background: none;
    color: #5a6674;
    width: 195px;
    height: 20px;
    border: none;
    appearance: none;
    outline: none;

    &::-webkit-search-cancel-button {
      appearance: none;
    }
  }

  .search-button {
    position: absolute;
    top: 10px;
    left: 15px;
    height: 20px;
    width: 20px;
    padding: 0;
    margin: 0;
    border: none;
    background: none;
    outline: none !important;
    cursor: pointer;

    & svg {
      width: 20px;
      height: 20px;
      fill: #5a6674;
    }
  }
}
</style>
