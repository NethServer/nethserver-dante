<template>
  <div>
    <h2>{{$t('dashboard.title')}}</h2>

    <iframe :src="url" frameborder="0"></iframe>
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
      url: ""
    };
  },
  methods: {
    getURL() {
      var context = this;
      nethserver.exec(
        ["nethserver-dante/read"],
        {
          action: "info"
        },
        null,
        function(success) {
          try {
            success = JSON.parse(success);
            context.view.isLoaded = true;
            context.url = success;
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
  width: 100%;
  height: 100%;
  border: none;
  margin-left: -20px;
  z-index: 1;
}
</style>
