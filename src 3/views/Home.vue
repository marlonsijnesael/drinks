<template>
  <div id="top" class="home">
    <v-app-bar
      dark
      flat
      color="#092327"
      absolute
      class="wave"
      hide-on-scroll
      scroll-target="#content"
    >
      Café Metro
      <v-spacer />
      <v-menu offset-y>
        <template v-slot:activator="{ on, attrs }">
          <v-app-bar-nav-icon v-bind="attrs" v-on="on"></v-app-bar-nav-icon>
        </template>
        <v-list dark>
          <v-list-item v-for="(item, index) in items" :key="index">
            <v-list-item-title @click="setComponentBanner(item.componentName)">{{
              item.title
            }}</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-app-bar>

    <div>
      <keep-alive>
        <component v-bind:is="pageOpen" />
      </keep-alive>
      <Bierlist v-if="show" />
      <FooterPart />
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import Bierlist from "@/components/Bierlijst.vue";
import Splash from "@/components/Splash.vue";
import Openings from "@/components/Openings.vue";
import DatePick from "@/components/DatePicker.vue";
import FooterPart from "@/components/Footer.vue";
export default {
  name: "Home",
  components: {
    Bierlist,
    Openings,
    Splash,
    DatePick,
    FooterPart,
  },

  data: () => ({
    allowed: false,
    dialog: true,
    show: true,
    showNav: false,
    pages: ["splash", "openings", "agenda"],
    pageOpen: "splash",
    items: [
      { title: "Home", componentName: "splash", icon: "mdi-home" },
      { title: "Openingstijden", componentName: "openings", icon: "mdi-clock" },
      { title: "Agenda", componentName: "splash", icon: "mdi-view-agenda" },
    ],
  }),
  methods: {
    setComponentBanner(component) {
      this.pageOpen = component;
      this.showNav = false;
      this.$vuetify.goTo("#waveBtm");
    },
  },
  computed: {
    showSite() {
      return this.$store.state.canEnter;
    },
  },
};
</script>

<style scoped>
.wave {
  height: 50px;
  position: relative;
}

.wave::before {
  content: "";
  position: absolute;
  left: 0;
  bottom: 0;
  right: 0;
  top: 50px;
  background-repeat: repeat;
  height: 20px;
  background-size: 20px 100px;
  background-image: radial-gradient(circle at 10px 5px, #092327 13px, transparent 12px);
}

.home {
  background-color: #092327;
}
</style>
