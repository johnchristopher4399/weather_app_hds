<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        您目前正在预览这个城市，点击"+"图标开始跟踪这个城市。
      </p>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{ weatherData.current.reporttime }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <i
        :class="['iconfont', weatherIconMap[weatherData.current.weather[0].icon] || 'icon-qing']"
        style="font-size:64px;"
      ></i>
      <p>湿度: {{ weatherData.current.humidity }}%</p>
      <p>风向: {{ weatherData.current.winddirection }}，风力: {{ weatherData.current.windpower }}级</p>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">未来3天天气预报</h2>
        <div
          v-for="(day, idx) in weatherData.daily.slice(0, 3)"
          :key="day.date"
          class="flex items-center"
        >
          <p class="flex-1">
            {{ day.date }} (周{{ day.week }})
          </p>
          <i
            :class="['iconfont', weatherIconMap[day.icon] || 'icon-qing']"
            style="font-size:32px;"
          ></i>
          <div class="flex gap-2 flex-1 justify-end">
            <p>白天: {{ day.dayweather }} {{ day.daytemp }}℃</p>
            <p>夜间: {{ day.nightweather }} {{ day.nighttemp }}℃</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const getWeatherData = async () => {
  try {
    // 获取当前天气
    const currentRes = await axios.get(
      `https://restapi.amap.com/v3/weather/weatherInfo?city=${route.query.adcode}&key=6bfdd730c233e3568a22b1488a2ef8fe&extensions=base`
    );
    // 获取预报天气
    const forecastRes = await axios.get(
      `https://restapi.amap.com/v3/weather/weatherInfo?city=${route.query.adcode}&key=6bfdd730c233e3568a22b1488a2ef8fe&extensions=all`
    );
    const current = currentRes.data.lives && currentRes.data.lives[0] ? currentRes.data.lives[0] : {};
    const forecast = forecastRes.data.forecasts && forecastRes.data.forecasts[0] ? forecastRes.data.forecasts[0] : {};
    // 组装数据结构
    return {
      current: {
        temp: current.temperature ? Number(current.temperature) : 0,
        weather: [
          {
            description: current.weather || "",
            icon: current.weather || ""
          }
        ],
        humidity: current.humidity,
        winddirection: current.winddirection,
        windpower: current.windpower,
        reporttime: current.reporttime,
      },
      daily: forecast.casts ? forecast.casts.map(day => ({
        date: day.date,
        week: day.week,
        dayweather: day.dayweather,
        nightweather: day.nightweather,
        daytemp: Number(day.daytemp),
        nighttemp: Number(day.nighttemp),
        daywind: day.daywind,
        nightwind: day.nightwind,
        daypower: day.daypower,
        nightpower: day.nightpower,
        icon: day.dayweather,
      })) : [],
      city: forecast.city,
      province: forecast.province,
      reporttime: forecast.reporttime,
    };
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();

const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updatedCities = cities.filter(
    (city) => city.id !== route.query.id
  );
  localStorage.setItem(
    "savedCities",
    JSON.stringify(updatedCities)
  );
  router.push({
    name: "home",
  });
};

// 天气图标映射
const weatherIconMap = {
  "晴": "icon-qing",
  "多云": "icon-duoyun",
  "阴": "icon-yin",
  "阵雨": "icon-zhenyu",
  "雷阵雨": "icon-leizhenyu",
  "雷阵雨伴有冰雹": "icon-leizhenyubingbao",
  "小雨": "icon-xiaoyu",
  "中雨": "icon-zhongyu",
  "大雨": "icon-dayu",
  "暴雨": "icon-baoyu",
  "大暴雨": "icon-dabaoyu",
  "特大暴雨": "icon-tedabaoyu",
  "阵雪": "icon-zhenxue",
  "小雪": "icon-xiaoxue",
  "中雪": "icon-zhongxue",
  "大雪": "icon-daxue",
  "暴雪": "icon-baoxue",
  "雨夹雪": "icon-yujiaxue",
  "冻雨": "icon-dongyu",
  "雾": "icon-wu",
  "霾": "icon-31mai",
  "浮尘": "icon-fuchen",
  "扬沙": "icon-yangsha",
  "强沙尘暴": "icon-qiangshachenbao",
  "小到中雨": "icon-xiaodaozhongyu",
  "中到大雨": "icon-zhongdaodayu",
  "大到暴雨": "icon-dadaobaoyu",
  "暴雨到大暴雨": "icon-baoyudaodabaoyu",
  "大暴雨到特大暴雨": "icon-dabaoyudaotedabaoyu",
  "小到中雪": "icon-xiaodaozhongxue",
  "中到大雪": "icon-zhongdaodaxue",
  "大到暴雪": "icon-dadaobaoxue",
};
</script>