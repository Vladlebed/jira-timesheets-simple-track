<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      work-exel-track
    </v-app-bar>

    <v-main>
      <exel-parser/>
    </v-main>
  </v-app>
</template>

<script>
import ExelParser from './components/ExelParser.vue';

export default {
  name: 'App',

  components: { ExelParser },
};
</script>
