<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
          type="text"
          v-model="searchQuery"
          @input="getSearchResults "
          placeholder="Search for a city or state"
          class="py-2 px-1 w-full bg-transparent border-b
          focus:border-weather-secondary
          focus: outline-none
          focus:shadow-[0px_1px_0_0_#004E71]"
      >
      <ul
          class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
          v-if="mapBoxSearchResults"
      >
        <p v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p v-if="!searchError && mapBoxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
              v-for="searchResult of mapBoxSearchResults"
              :key="searchResult.id"
              class="py-2 cursor-pointer"
              @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
  import {useRouter} from "vue-router";

  const mapboxAPIKey = "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";

  import {ref} from "vue";
  import axios from "axios";
  import CityList from "@/components/CityList.vue";
  import CityCardSkeleton from "@/components/CityCardSkeleton.vue";

  const router = useRouter();

  const previewCity = (searchResult) => {
    const [city, state] = searchResult.place_name.split(',')
    router.push({
      name: "cityView",
      params: { city, state: state.replaceAll(" ", "") },
      query: {
        lat: searchResult.geometry.coordinates[1],
        lng: searchResult.geometry.coordinates[0],
        preview: true
      },
    });
  };

  const searchQuery = ref("");
  const queryTimeout = ref(null);
  const mapBoxSearchResults = ref(null);
  const searchError = ref(null);

  const getSearchResults = () => {
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout(async () => {
      if (searchQuery.value !== "") {
        try {
          const result = await axios.get(
              `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
          );
          mapBoxSearchResults.value = result.data.features
        } catch (e) {
          searchError.value = true;
        }
        return;
      }

      mapBoxSearchResults.value = null;
    }, 300)
  };
</script>
