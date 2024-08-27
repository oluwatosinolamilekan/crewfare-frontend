
<template>
    <div>
     
    <h1>Top 10 Countries by Population</h1>
    <select v-model="selectedRegion" @change="fetchCountriesByRegion">
      <option value="">All Regions</option>
      <option v-for="region in regions" :key="region" :value="region">
        {{ region }}
      </option>
    </select>
    <ul>
      <li v-for="country in countries" :key="country.name.common">
        <img :src="country.flags.png" alt="Flag" width="50">
        <strong>{{ country.name.common }}</strong> - Population: {{ country.population }} - Capital: {{ country.capital }}
      </li>
    </ul>
  </div>
</template>


<script>
  import axios from 'axios';
    export default {
        data() {
            return {
            countries: [],
            selectedRegion: '',
            regions: ['Africa', 'Americas', 'Asia', 'Europe', 'Oceania']
            };
    },
    methods: {
        fetchCountries() {
        axios.get('http://localhost:8000/api/countries')
            .then(response => {
            this.countries = response.data;
            });
        },
        fetchCountriesByRegion() {
        let url = 'http://localhost:8000/api/countries';
        if (this.selectedRegion) {
            url += `/region/${this.selectedRegion}`;
        }
        axios.get(url)
            .then(response => {
            this.countries = response.data;
            });
        }
    },
    created() {
            this.fetchCountries();
    }
};
</script>

<style scoped>

</style>