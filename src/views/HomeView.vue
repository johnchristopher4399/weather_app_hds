<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="搜索城市或省份"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="amapSearchResults"
      >
        <p class="py-2" v-if="searchError">
          对不起，出错了，请再试一次。
        </p>
        <p
          class="py-2"
          v-if="!searchError && amapSearchResults.length === 0"
        >
          没有结果匹配您的查询，请尝试不同的术语。
        </p>
        <template v-else>
          <li
            v-for="(searchResult, idx) in amapSearchResults"
            :key="searchResult.adcode + '-' + searchResult.city + '-' + idx"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.city }} {{ searchResult.province }}
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
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";
import CityList from "../components/CityList.vue";

const router = useRouter();
const previewCity = (searchResult) => {
  router.push({
    name: "cityView",
    params: { state: searchResult.province || '直辖市', city: searchResult.city },
    query: {
      adcode: searchResult.adcode,
      preview: true,
    },
  });
};

const amapKey = "6bfdd730c233e3568a22b1488a2ef8fe";
const searchQuery = ref("");
const queryTimeout = ref(null);
const amapSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        // 高德行政区查询接口，返回城市及adcode
        const result = await axios.get(
          `https://restapi.amap.com/v3/config/district?keywords=${searchQuery.value}&subdistrict=2&key=${amapKey}`
        );
        // 只取城市级别的结果
        let cities = [];
        if (result.data.districts && result.data.districts.length > 0) {
          result.data.districts.forEach((item) => {
            if (item.level === "province" && item.districts) {
              // 省下属城市
              item.districts.forEach((city) => {
                if (city.level === "city") {
                  cities.push({
                    city: city.name,
                    province: item.name,
                    adcode: city.adcode,
                  });
                }
              });
            } else if (item.level === "city") {
              // 兼容直辖市等没有province字段的情况
              cities.push({
                city: item.name,
                province: item.province || item.name,
                adcode: item.adcode,
              });
            }
          });
        }
        amapSearchResults.value = cities.slice(0, 10);
        searchError.value = false;
      } catch {
        searchError.value = true;
      }
      return;
    }
    amapSearchResults.value = null;
  }, 300);
};
</script>

<style lang="scss" scoped></style>