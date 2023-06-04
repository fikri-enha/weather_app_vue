<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No location added. to start tracking a location, search in the field above
  </p>
</template>
<script setup>
import { callWithAsyncErrorHandling, ref } from "vue";
import CityCard from "./CityCard.vue";
import axios from "axios";
import { useRouter } from "vue-router";
const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
  }

  // get weatherdata for each cities
  const requests = [];
  savedCities.value.forEach((city) => {
    requests.push(
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=f3533bd5c5626313f66069b7183ae0e5&units=imperial`
      )
    );
  });

  // promising the async call api
  const weatherData = await Promise.all(requests);

  // Flicker delay
  await new Promise((res) => setTimeout(res, 1000));

  // adding weather value to each saved cities by index
  weatherData.forEach((value, index) => {
    savedCities.value[index].weather = value.data;
  });
};

await getCities();
const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: {
      state: city.state,
      city: city.city,
    },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
    },
  });
};
</script>
