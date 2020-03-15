<template>
  <v-app>
    <v-content>
      <Covid19/>
    </v-content>
  </v-app>
</template>

<script>
import Covid19 from './components/Covid19';

export default {
  name: 'App',

  components: {
    Covid19,
  },

  data: () => ({
    //
  }),
};
</script>
