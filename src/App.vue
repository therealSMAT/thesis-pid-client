<template>
  <div id="app">
    <div class="container">
      <div class="header">
        <img
          src="https://upload.wikimedia.org/wikipedia/en/3/3d/Tallinna_Linnatranspordi_AS_logo.png"
          alt="tlt logo"
          width="64"
        />
        <h1>Tallinna Linnatranspordi</h1>
      </div>
      <div class="item">
        <div id="timeline" v-if="routes.length">
          <div >
            <section
              class="date"
              v-for="{ time, location_name, tripId } in routes"
              :key="location_name"
            >
              <h3>{{ time }}</h3>
              <section>
                <ul :class="{'active': currentStop === tripId}">
                  <li>{{ location_name }}</li>
                </ul>
              </section>
            </section>
          </div>
        </div>
        <div v-else>
          <h4>No route information!</h4>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import mqtt from "mqtt";

export default {
  name: "App",
  data() {
    return {
      client: null,
      routes: [],
      currentStop: null,
    };
  },
  mounted() {
    this.createConnection();
  },
  computed: {
    clientId() {
      return 'mqttjs_' + Math.random().toString(16).substr(2, 8)
    },
  },
  methods: {
    createConnection() {
      this.client = mqtt.connect(process.env.VUE_APP_MQTT_BROKER, { clientId: this.clientId });

      this.subscribeToTopics();
      this.handleErrors();
      this.waitForMessages();
    },
    handleErrors() {
      this.client.on("error", (err) => {
        console.log("Connection error: ", err);
        this.client.end();
      });
    },
    subscribeToTopics() {
      this.client.on("connect", () => {
        this.client.subscribe("transport/ride/start");
        this.client.subscribe("transport/currentStop");
        this.client.subscribe("transport/ride/stop");
        console.log("Subscribed successfully!");
      });
    },
    waitForMessages() {
      this.client.on("message", (topic, message) => {
        if (topic === "transport/ride/start") {
          let { data } = JSON.parse(message.toString());
          this.$set(this, "routes", data);
        }

        if (topic === "transport/currentStop") {
          this.currentStop = message.toString();
        }

        if (topic === 'transport/ride/stop') {
          this.$set(this, "routes", []);
          this.currentStop = null;
        }
      });
    },
  },
};
</script>

<style>
@charset "UTF-8";
@import url(https://fonts.googleapis.com/css?family=Fira+Sans:200,400,500);
* {
  border: 0;
  margin: 0;
  padding: 0;
}

html {
  height: 100%;
}

body {
  height: inherit;
  display: flex;
  flex-direction: column;
  font-family: "Fira Sans", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #79838c;
}

a {
  color: #50585f;
  text-decoration: none;
}
a:hover {
  color: #383e44;
}

div.container {
  display: flex;
  flex: auto;
  flex-direction: column;
  max-height: 100%;
}

div.header {
  height: auto;
  text-align: center;
  background: slategrey;
  color: ghostwhite;
  padding: 2.3rem 1rem 2.3rem 1rem;
  position: relative;
}
div.header:after {
  content: "";
  position: absolute;
  bottom: -5rem;
  left: 0rem;
  height: 5.1rem;
  display: block;
  width: 100%;
  z-index: 300;
  background: -moz-linear-gradient(top, white 20%, rgba(255, 255, 255, 0) 100%);
  /* FF3.6-15 */
  background: -webkit-linear-gradient(
    top,
    white 20%,
    rgba(255, 255, 255, 0) 100%
  );
  /* Chrome10-25,Safari5.1-6 */
  background: linear-gradient(
    to bottom,
    white 20%,
    rgba(255, 255, 255, 0) 100%
  );
  /* W3C, IE10+, FF16+, Chrome26+, Opera12+, Safari7+ */
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr="#ffffff", endColorstr="#00ffffff",GradientType=0 );
  /* IE6-9 */
}
div.header h1 {
  margin-top: 0.8rem;
  margin-bottom: 0.5rem;
  font-weight: 200;
  font-size: 1.6em;
  letter-spacing: 0.1rem;
  text-transform: uppercase;
}
@media (min-width: 62em) {
  div.header h1 {
    font-size: 1.9em;
    letter-spacing: 0.2rem;
  }
}
div.header h2 {
  font-size: 1.1em;
  font-weight: 400;
  color: #cfd7de;
  max-width: 30rem;
  margin: auto;
}

div.item {
  display: flex;
  flex: auto;
  overflow-y: auto;
  padding: 0rem 1rem 0rem 1rem;
}

#timeline {
  position: relative;
  display: table;
  height: 100%;
  margin-left: auto;
  margin-right: auto;
  margin-top: 5rem;
}
#timeline div:after {
  content: "";
  width: 2px;
  position: absolute;
  top: 0.5rem;
  bottom: 0rem;
  left: 60px;
  z-index: 1;
  background: #c5c5c5;
}
#timeline h3 {
  position: -webkit-sticky;
  position: sticky;
  top: 5rem;
  color: #888;
  margin: 0;
  font-size: 1em;
  font-weight: 400;
}
@media (min-width: 62em) {
  #timeline h3 {
    font-size: 1.1em;
  }
}
#timeline section.date {
  position: relative;
}
#timeline section.date:first-child section {
  margin-top: -1.3em;
  padding-bottom: 0px;
}
#timeline section.date section {
  position: relative;
  padding-bottom: 1.25em;
  margin-bottom: 2.2em;
}
#timeline section.date section h4 {
  position: absolute;
  bottom: 0;
  font-size: 0.9em;
  font-weight: 400;
  line-height: 1.2em;
  margin: 0;
  padding: 0 0 0 89px;
  color: #c5c5c5;
}
@media (min-width: 62em) {
  #timeline section.date section h4 {
    font-size: 1em;
  }
}
#timeline section.date section ul {
  list-style-type: none;
  padding: 0 0 0 75px;
  margin: -1.35rem 0 1em;
  max-width: 32rem;
  font-size: 1em;
}
@media (min-width: 62em) {
  #timeline section.date section ul {
    font-size: 1.1em;
    padding: 0 0 0 81px;
  }
}
#timeline section.date section ul:last-child {
  margin-bottom: 0;
}
#timeline section.date section ul:first-of-type:after {
  content: "";
  width: 10px;
  height: 10px;
  background: #c5c5c5;
  border: 2px solid #ffffff;
  -webkit-border-radius: 50%;
  -moz-border-radius: 50%;
  -ms-border-radius: 50%;
  border-radius: 50%;
  position: absolute;
  left: 54px;
  top: 3px;
  z-index: 2;
}
#timeline section.date section ul.active:first-of-type:after {
  background: #30a757;
  animation-name: pulse-ring;
  animation-duration: 2s;
  animation-iteration-count: infinite;
}
#timeline section.date section ul.active li {
  color: #30a757;
  font-weight: 800;
}
@keyframes pulse-ring {
  0% {
    border-color: rgba(255, 255, 255, 0.5);
    transform: scale(1);
    box-shadow: 2px 2px 10px 0 #fffbfb, inset 2px 2px 10px #fffbfb;
  }
  25% {
    transform: scale(0.98);
    box-shadow: 2px 2px 10px 2.5px #fffbfb, inset 2px 2px 12px #fffbfb;
  }
  50% {
    border-color: rgba(255, 255, 255, 0.75);
    transform: scale(1);
    box-shadow: 2px 2px 10px 5px #fffbfb, inset 2px 2px 10px #fffbfb;
  }
  75% {
    transform: scale(0.98);
    box-shadow: 2px 2px 10px 2.5px #fffbfb, inset 2px 2px 12px #fffbfb;
  }
  100% {
    border-color: rgba(255, 255, 255, 0.5);
    transform: scale(1);
    box-shadow: 2px 2px 10px 0 #fffbfb, inset 2px 2px 10px #fffbfb;
  }
}
#timeline section.date section ul li {
  margin-left: 0.5rem;
}
#timeline section.date section ul li:before {
  content: "·";
  margin-left: -0.5rem;
  padding-right: 0.3rem;
}
#timeline section.date section ul li:not(:first-child) {
  margin-top: 0.5rem;
}
#timeline section.date section ul li span.price {
  color: mediumturquoise;
  font-weight: 500;
}

.usd {
  display: inline;
}

svg {
  border: 3px solid white;
  border-radius: 50%;
  box-shadow: 0 4px 6px rgba(50, 50, 93, 0.11), 0 1px 3px rgba(0, 0, 0, 0.08);
}
</style>
