<template>
  <div>
    <vue-json-pretty :path="'res'" :data="graphData"> </vue-json-pretty>
  </div>
</template>
<script>
import VueJsonPretty from 'vue-json-pretty';
import 'vue-json-pretty/lib/styles.css';

export default {
  props: {
    graphData: String
  },
  components: {
    VueJsonPretty,
  },
};
</script>
<style scoped>
</style>
