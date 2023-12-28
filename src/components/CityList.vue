<template>
  <div v-for="city of savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>
  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a location, search in
    the field above.
  </p>
</template>

<script setup>
import {ref} from "vue";
import axios from "axios";
import CityCard from "@/components/CityCard.vue";
import {useRouter} from "vue-router";

const router = useRouter();

const goToCityView = (city) => {
  const { id, city: cityParams, state, coords: { lat, lng } } = city;

  router.push({
    name: 'cityView',
    params: { city: cityParams, state },
    query: { id, lat, lng }
  });
};
const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'))
  }

  const requests = [];

  savedCities.value.forEach((city) => {
    requests.push(
        axios.get(
            `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
        )
    );
  })

  const weatherData = await Promise.all(requests);

  weatherData.forEach((value, index) => {
    savedCities.value[index].weather = value.data;
  })
};
await getCities();
</script>
