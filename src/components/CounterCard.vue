<template>
  <v-card class="mx-auto" max-width="344" :loading="pending" :color="color" elevation="15">
    <v-card-text>
      <p class="cardName text--primary"></p>
      <h1>{{name}}</h1>
      <div class="cardData display-2 text--primary">{{data}}</div>
    </v-card-text>
  </v-card>
</template>

<script>
import axios from "axios";

export default {
  name: "CounterCard",
  props: {
    name: { type: String, required: true },
    api: { type: String, required: true },
    color: { type: String, default: null, require: false },
    countryRegion: { type: String, default: null, require: false },
    typeData: { type: String, required: true }
  },
  data() {
    return {
      pending: true,
      errorCnt: 0,
      data: 0
    };
  },
  mounted() {
    this.requestData();
    this.countDown();
  },
  watch: {
    countryRegion: function() {
      this.requestData();
    }
  },
  methods: {
    async requestData() {
      this.pending = true;
      try {
        const { data } = await axios.get(this.api);
        this.data = 0;
        if (this.countryRegion) {
          data
            .filter(({ countryRegion }) => countryRegion == this.countryRegion)
            .forEach(el => {
              this.data += el[this.typeData];
            });
        } else {
          data.forEach(el => {
            this.data += el[this.typeData];
          });
        }
        this.errorCnt = 0;
      } catch (e) {
        this.data = 0;
        this.errorCnt += 1;
      }
      this.pending = false;
    },
    countDown() {
      setInterval(() => {
        this.requestData();
      }, 60000);
    }
  }
};
</script>

<style scoped>
.cardName {
  /* font-family: 'Poppins', sans-serif; */
  font-family: "Ubuntu", sans-serif;
}
.cardData {
  text-align: right;
}
</style>