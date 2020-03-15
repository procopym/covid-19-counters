<template>
  <v-item-group mandatory>
    <v-container grid-list-xl text-xs-center>
      <v-layout row wrap>
        <v-flex xs6 offset-xs3>
          <v-select label="Countries" v-if="countries.length>0" 
            :items="countries" 
            @change="this.onChange"
            @click:clear="this.onReset"
            clearable></v-select>
        </v-flex>
      </v-layout>
    </v-container>
    <v-container>
      <v-row>
        <v-col v-for="card in cards" :key="card.name" cols="12" md="4">
          <CounterCard
            :name="card.name"
            :api="card.api"
            :color="card.color"
            :countryRegion="countryRegion"
            :typeData="card.typeData"
          />
        </v-col>
      </v-row>
    </v-container>
  </v-item-group>
</template>

<script>
import axios from "axios";
import CounterCard from "./CounterCard";

export default {
  name: "Covid19",
  components: {
    CounterCard
  },
  mounted(){
    this.requestCountries();
  },
  data: () => ({
    countryRegion: null,
    countries: [],
    cards: [
      {
        name: "Confirmed",
        api: "https://covid19.mathdro.id/api/confirmed",
        typeData: "confirmed",
        color: "#2196F3" //Blue
      },
      {
        name: "Recovered",
        api: "https://covid19.mathdro.id/api/recovered",
        typeData: "recovered",
        color: "#4CAF50" //Green
      },
      {
        name: "Deaths",
        api: "https://covid19.mathdro.id/api/deaths",
        typeData: "deaths",
        color: "#F44336" //Stong red
      },
      {
        name: "Active",
        api: "https://covid19.mathdro.id/api/confirmed",
        typeData: "active",
      }
    ]
  }),
  methods: {
    onReset() {
      this.countryRegion = null;
      return true
    },
    async requestCountries() {
      try {
        const { data } = await axios.get("https://covid19.mathdro.id/api/countries");
          this.countries = Object.keys(data.countries);
      } catch (e) {
        console.log(e)
      }
    },
    onChange(e){
      this.countryRegion = e;
    }
  }
};
</script>

<style scoped>
.resetBtnBox {
  align-self: center;
  text-align: center;
}
</style>