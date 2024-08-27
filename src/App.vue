<template>
  <div class="container-fluid">
    <h1>Top 10 Countries by Population</h1>
    <div class="row mb-3">
      <div class="col-md-4">
        <input
          type="text"
          v-model="searchTerm"
          @input="filterCountries"
          class="form-control mb-2"
          placeholder="Search by region..."
        />
        <select v-model="selectedRegion" @change="fetchCountriesByRegion" class="form-select mb-2">
          <option value="">All Regions</option>
          <option v-for="region in regions" :key="region" :value="region">
            {{ region }}
          </option>
        </select>
      </div>
    </div>
    <div v-if="filteredCountries.length === 0" class="alert alert-info">
      No results found.
    </div>
    <div v-else class="table-responsive">
      <table class="table table-striped">
        <thead>
          <tr>
            <th @click="sortTable('name')">Name</th>
            <th @click="sortTable('population')">Population</th>
            <th>Capital</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="country in paginatedCountries" :key="country.name">
            <td>
              <img :src="country.flag" alt="Flag" width="50" class="me-2">
              {{ country.name }}
            </td>
            <td>{{ country.population.toLocaleString() }}</td>
            <td>{{ formatCapital(country.capital) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
 
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      countries: [],
      filteredCountries: [],
      searchTerm: '',
      selectedRegion: '',
      regions: ['Africa', 'Americas', 'Asia', 'Europe', 'Oceania'],
      currentPage: 1,
      itemsPerPage: 10,
      sortKey: 'name', // Default sort key
      sortOrder: 'asc' // Default sort order
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.filteredCountries.length / this.itemsPerPage);
    },
    paginatedCountries() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.sortedCountries.slice(start, end);
    },
    sortedCountries() {
      return this.filteredCountries.slice().sort((a, b) => {
        if (this.sortKey === 'population') {
          return this.sortOrder === 'asc'
            ? a[this.sortKey] - b[this.sortKey]
            : b[this.sortKey] - a[this.sortKey];
        }
        if (a[this.sortKey] < b[this.sortKey]) return this.sortOrder === 'asc' ? -1 : 1;
        if (a[this.sortKey] > b[this.sortKey]) return this.sortOrder === 'asc' ? 1 : -1;
        return 0;
      });
    }
  },
  methods: {
    fetchCountriesByRegion() {
      let url = 'http://127.0.0.1:8000/countries';
      if (this.selectedRegion) {
        url += `/region/${this.selectedRegion}`;
      }
      axios.get(url)
        .then(response => {
          this.countries = response.data;
          this.filterCountries(); 
        });
    },
    filterCountries() {
      let filtered = this.countries;
      if (this.searchTerm) {
        filtered = filtered.filter(country =>
          (country.region || '').toLowerCase().includes(this.searchTerm.toLowerCase())
        );
      }
      this.filteredCountries = filtered;
      this.currentPage = 1; // Reset to the first page after filtering
    },
    sortTable(key) {
      if (this.sortKey === key) {
        this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
      } else {
        this.sortKey = key;
        this.sortOrder = 'asc';
      }
    },
    goToPage(page) {
      if (page < 1 || page > this.totalPages) return;
      this.currentPage = page;
    },
    formatCapital(capital) {
      if (Array.isArray(capital)) {
        return capital.join(', ');
      }
      return capital;
    }
  },
  watch: {
    searchTerm() {
      this.filterCountries();
    },
    selectedRegion() {
      this.fetchCountriesByRegion(); // Fetch data when the selected region changes
    },
    itemsPerPage() {
      this.filterCountries(); // Reapply filters when items per page changes
    }
  },
  created() {
    this.fetchCountriesByRegion(); // Fetch initial data
  }
};
</script>

<style scoped>
.table img {
  max-width: 100%;
  height: auto;
}
.pagination .page-item.disabled .page-link {
  pointer-events: none;
  opacity: 0.5;
}
.alert-info {
  text-align: center;
}
</style>
