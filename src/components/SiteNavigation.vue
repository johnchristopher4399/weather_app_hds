<template>
  <header class="sticky top-0 bg-weather-primary shadow-lg">
    <nav
      class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6"
    >
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3">
          <i class="fa-solid fa-sun text-2xl"></i>
          <p class="text-2xl">当地天气</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="toggleModal"
        ></i>
        <i
          class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="addCity"
          v-if="route.query"
        ></i>
      </div>

      <BaseModal
        :modalActive="modalActive"
        @close-modal="toggleModal"
      >
        <div class="text-black">
          <h1 class="text-2xl mb-3">关于：</h1>
          <p class="mb-4">
            本地天气允许您跟踪当前和您选择的城市的未来天气。
          </p>
          <h2 class="text-2xl mb-3">它是如何生效的：</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>
              通过在搜索栏中输入名称来搜索您的城市。
            </li>
            <li>
              在结果中选择一个城市，这将带你到当前天气为你选择。
            </li>
            <li>
              通过点击右上角的"+"图标来追踪城市。这将保存城市，以便稍后在主页上查看。
            </li>
          </ol>

          <h2 class="text-2xl mb-3">移除城市</h2>
          <p>
            如果您不再希望跟踪某个城市，只需在主页中选择该城市即可。在页面底部，将有一个删除城市的选项。
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";

const savedCities = ref([]);
const route = useRoute();
const router = useRouter();
const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities")
    );
  }

  const locationObj = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    adcode: route.query.adcode,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  };

  savedCities.value.push(locationObj);
  localStorage.setItem(
    "savedCities",
    JSON.stringify(savedCities.value)
  );

  let query = Object.assign({}, route.query);
  delete query.preview;
  query.id = locationObj.id;
  router.replace({ query });
};

const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>