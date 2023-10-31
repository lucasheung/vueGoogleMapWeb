<template>
  <router-view />
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "App",
  methods: {},
  computed: {
    apiKey() {
      return this.$store.state.moduleExample.apiKey;
    },
  },
  mounted() {
    let recaptchaScript = document.createElement("script");
    recaptchaScript.setAttribute(
      "src",
      `https://maps.googleapis.com/maps/api/js?key=${this.apiKey}&libraries=places`
    );
    document.head.appendChild(recaptchaScript);
  },
});
</script>
