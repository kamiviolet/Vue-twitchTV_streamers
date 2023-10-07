<script>
const channels = [
  "ESL_SC2",
  "OgamingSC2",
  "cretetion",
  "freecodecamp",
  "storbeck",
  "habathcx",
  "RobotCaleb",
  "noobs2ninjas"
];
const proxy = "https://twitch-proxy.freecodecamp.rocks/twitch-api/";

export default {
  data() {
    return {
      active: "all",
      message: "Twitch Streamers",
      status: ["all", "online", "offline"],
      listOfChannels: []
    };
  },
  methods: {
    async fetchInfo() {
      this.listOfChannels = [];
      try {
        Promise.all(
          channels.map((ch) => {
            fetch(proxy + "channels/" + ch)
              .then((res) => res.json())
              .then((data) => {
                return {
                  id: ch,
                  logo:
                    data.logo ??
                    "https://dummyimage.com/50x50/ecf0e7/5c5457.jpg&text=0x3F",
                  name: data.display_name ?? ch,
                  description: data.status,
                  url: data.url,
                  game: data.game
                };
              })
              .then((d) => {
                return fetch(proxy + "streams/" + ch)
                  .then((res) => res.json())
                  .then((data) => {
                    const entry = { game: "", status: "" };
                    const stream = data.stream;
                    if (stream === null) {
                      entry.game = "Offline";
                      entry.status = "offline";
                    } else if (stream === undefined) {
                      entry.game = "Account Closed";
                      entry.status = "offline";
                    } else {
                      entry.game = data.stream.game;
                      entry.status = "online";
                    }
                    return { ...entry, ...d };
                  });
              })
              .then((result) => {
                this.listOfChannels.push(result);
              });
          })
        );
      } catch (err) {
        console.log(err.message);
      }
    },
    changeActive(stat) {
      this.active = stat;
    }
  },
  mounted() {
    this.fetchInfo();
  },
  computed: {
    filterList() {
      if (this.active == "online") {
        return this.listOfChannels.filter((ch) => ch.status == "online");
      } else if (this.active == "offline") {
        return this.listOfChannels.filter((ch) => ch.status != "online");
      } else {
        return this.listOfChannels;
      }
    }
  }
};
</script>

<template>
  <div id="app">
    <header>
      <h1>{{ message }}</h1>
      <section class="menu">
        <div v-for="stat of status" class="selector" :id="stat">
          <div class="circle"></div>
          <button @click.prevent="changeActive(stat)">{{ stat }}</button>
        </div>
      </section>
    </header>
    <div id="display">
      <div v-for="entry of filterList" class="entry">
        <div class="thumbnail">
          <div class="thumbnail_wrapper">
            <img :src="entry.logo" />
            <div :class="entry.status" class="status"></div>
          </div>
        </div>
        <div class="details">
          <a :href="entry.url">
            <p>{{ entry.name }}</p>
          </a>
          <em v-if="entry.status == 'online'">
            {{ entry.game }}: {{ entry.description }}
          </em>
          <p v-else>offline</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
header {
  display: flex;
  justify-content: space-between;
}

h1 {
  text-align: left;
  text-transform: uppercase;
}

.menu {
  float: right;
  font-size: 0.8em;
  font-weight: bold;
  text-transform: uppercase;
  padding: 1em;
  text-align: right;
  margin-inline-start: 2em;
}

a,
button {
  color: #5a1073;
  text-decoration: none;
}

a {
  font-weight: 800;
  margin: 0;
  padding: 0;
}

button {
  background: none;
  width: 75px;
  border: none;
  padding-block: 1em;
  cursor: pointer;
  font-weight: 800;

  &:hover {
    background-color: orange;
    color: white;
  }
}

#display {
  display: grid;
  justify-items: center;
  width: 100%;

  @media (width > 768px) {
    grid-template-columns: repeat(auto-fill, 275px);
  }
}

.entry {
  display: grid;
  grid-template-rows: auto;
  justify-items: first-baseline;
  align-items: first-baseline;
  margin: 1em;
  background-color: rgb(255, 255, 255, 0.7);
  padding: 1em;
  width: inherit;
  grid-template-rows: 120px auto;

  @media (width > 768px) {
    width: 225px;
    min-height: auto;
  }
}

.thumbnail {
  justify-self: center;
  width: 125px;
  position: relative;

  @media (width > 480px) {
    justify-self: first-baseline;
  }

  img {
    max-width: 100px;
    width: auto;
    object-fit: contain;
    height: auto;
  }
}

.thumbnail_wrapper {

  background-color: #333;
  border: 5px white solid;
  border-radius: 50%;
  width: 100px;
  overflow: hidden;
  height: 100px;
}

.details {
  justify-self: stretch;
}

.status {
  width: 25px;
  height: 25px;
  background: #ccc;
  position: absolute;
  border-radius: 50%;
  bottom: 5px;
  right: 25%;
}

.online {
  background: green;
}
</style>
