<template>
  <div>
      <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)" />
      </div>

      <p v-if="savedCities.length === 0">
        没有新增地点。要开始跟踪位置，请在上面的字段中搜索。
      </p>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";
import { useRouter } from "vue-router";
import CityCard from "./CityCard.vue";

const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities")
    );

    const requests = [];
    savedCities.value.forEach((city) => {
      requests.push(
        axios.get(
          `https://restapi.amap.com/v3/weather/weatherInfo?city=${city.adcode}&key=6bfdd730c233e3568a22b1488a2ef8fe&extensions=base`
        )
      );
    });

    const weatherData = await Promise.all(requests);

    weatherData.forEach((value, index) => {
      const live = value.data.lives && value.data.lives[0] ? value.data.lives[0] : {};
      savedCities.value[index].weather = {
        main: {
          temp: live.temperature ? Number(live.temperature) : 0,
          temp_max: live.temperature ? Number(live.temperature) : 0,
          temp_min: live.temperature ? Number(live.temperature) : 0,
        },
        weather: [
          {
            description: live.weather || "",
            icon: live.weather || ""
          }
        ]
      };
    });
  }
};
await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { state: city.state || '直辖市', city: city.city },
    query: {
      adcode: city.adcode,
      id: city.id,
    },
  });
};
</script>