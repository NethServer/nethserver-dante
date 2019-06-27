<template>
  <div>
    <div v-if="!view.isLoaded" class="spinner spinner-lg"></div>
    <iframe :width="width" v-show="view.isLoaded" :src="url" frameborder="0"></iframe>
  </div>
</template>

<script>
export default {
  name: "Dashboard",
  mounted() {
    this.getURL();
  },
  data() {
    return {
      view: {
        isLoaded: false
      },
      url: "",
      width: window.innerWidth - 70
    };
  },
  methods: {
    getURL() {
      var context = this;
      nethserver.exec(
        ["nethserver-dante/dashboard/read"],
        {
          hostname: window.location.hostname
        },
        null,
        function(success) {
          try {
            success = JSON.parse(success);
            context.view.isLoaded = true;
            context.url = success.url;
          } catch (e) {
            console.error(e);
            context.view.isLoaded = true;
          }
        },
        function(error) {
          console.error(error);
        }
      );
    }
  }
};
</script>

<style>
iframe {
  position: absolute;
  height: 100%;
  border: none;
  margin-left: -20px;
  z-index: 1;
}
</style>
