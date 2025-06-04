<template>
  <div
    class="flex py-6 px-3 bg-weather-secondary rounded-md shadow-md cursor-pointer mb-4"
  >
    <div class="flex flex-col flex-1">
      <h2 class="text-3xl mb-2">{{ city.city }}</h2>
      <h3>{{ city.state }}</h3>
    </div>

    <div class="flex flex-col gap-2">
      <p class="text-3xl self-end">
        {{ Math.round(city.weather.main.temp) }}&deg;
      </p>
      <div class="flex gap-2 items-center">
        <span class="text-xs">
          H:
          {{ Math.round(city.weather.main.temp_max) }}&deg;
        </span>
        <span class="text-xs">
          L:
          {{ Math.round(city.weather.main.temp_min) }}&deg;
        </span>
        <i v-if="city.weather.weather[0].icon" :class="['iconfont', weatherIconMap[city.weather.weather[0].icon] || 'icon-qing']" style="font-size:32px;"></i>
        <span class="text-xs">{{ city.weather.weather[0].description }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
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

defineProps({
  city: {
    type: Object,
    default: () => {},
  },
});
</script>