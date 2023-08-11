<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="search"
        @input="getSeachResults"
        v-model="searchQuery"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004e71]"
      />
      <ul
        v-if="mapboxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">Sorry, something went wrong. Please try again.</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <!-- Cannot use a v-for and v-else directive on the same element, 
            so we wrapped our li tag in a template tag and put the v-else on the template tag instead. -->
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const mapboxAPIKey =
  "pk.eyJ1IjoiaXRzY2VzcyIsImEiOiJjbGw1dWI2bncwZ3lvM2pwODZoMnd1Z29lIn0.vc5kHVOmqmCVcdMy85CKPw";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);

const router = useRouter();

const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.place_name.split(",");
  router.push({
    name: "cityView",
    //replaceAll replaces the space before the state with an empty string
    //params match the params we gave this path in the router file. setting them to the searchResults.
    params: { state: state.replaceAll(" ", ""), city: city },
    //sets the query params in the URL to searchResults
    query: {
      lat: searchResult.geometry.coordinates[1],
      long: searchResult.geometry.coordinates[0],
      preview: true
    }
  });
};

const getSeachResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }
      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};
</script>
