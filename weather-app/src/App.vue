<template>
  <div :class="['weather-app', currentWeather]">
    <div class="weather-card">
      
      <input
        v-model="cityInput"
        @keyup.enter="fetchWeather"
        placeholder="都市名を英語表記で入力"
        class="search-box"
      />
      <button @click="fetchWeather">検索</button>

      
      <div v-if="weather">
        <h1>{{ city }}</h1>
        <div class="temp">{{ weather.main.temp }}°C</div>
        <p>{{ weather.weather[0].description }}</p>
      </div>
      <div class="icon">
          <i :class="weatherIcon"></i>
        </div>
      <p v-if="error" style="color: red;">{{ error }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const API_KEY = "6a5cf78c3eb124b2cf1f16967b88a351"; 

const cityInput = ref("Tokyo");
const city = ref("Tokyo"); 
const weather = ref(null);
const error = ref(null);
const lastWeather = ref("");

const currentWeather = computed(() => {
  if (!weather.value) return lastWeather.value; 
  const main = weather.value.weather[0].main;
  let newWeather = "";
  switch (main) {
    case "Clear": newWeather = "sunny"; break;
    case "Clouds": newWeather = "cloudy"; break;
    case "Rain": newWeather = "rainy"; break;
    case "Snow": newWeather = "snowy"; break;
    case "Thunderstorm": newWeather = "stormy"; break;
    default: newWeather = lastWeather.value; 
  }
  lastWeather.value = newWeather; 
  return newWeather;
});

const weatherIcon = computed(() => {
  if (!weather.value) return "fas fa-question-circle";
  const main = weather.value.weather[0].main;
  switch (main) {
    case "Clear": return "fas fa-sun";
    case "Clouds": return "fas fa-cloud";
    case "Rain": return "fas fa-cloud-showers-heavy";
    case "Snow": return "fas fa-snowflake";
    case "Thunderstorm": return "fas fa-bolt";
    default: return "";
  }
});

// 天気取得
const fetchWeather = async () => {
  try {
    error.value = null;
    const url = `https://api.openweathermap.org/data/2.5/weather?q=${cityInput.value}&appid=${API_KEY}&units=metric&lang=ja`;
    const res = await fetch(url);
    const data = await res.json();

    if (data.cod !== 200) {
      error.value = "天気情報を取得できませんでした";
      weather.value = null;
      return;
    }
    
    weather.value = data;
    city.value = data.name;
  } catch (err) {
    error.value = "天気情報を取得できませんでした";
    console.error(err);
  }
};

// 初期表示（Tokyo）
fetchWeather();
</script>

<style>
/* 検索欄 */
.search-box {
  width: 80%;
  padding: 8px;
  margin-bottom: 10px;
  border-radius: 8px;
  border: none;
  font-size: 16px;
}

button {
  margin-bottom: 20px;
  padding: 8px 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  background: rgba(255,255,255,0.6);
  transition: background 0.3s;
}
button:hover {
  background: rgba(255,255,255,0.9);
}

/* 全体 */
.weather-app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  transition: background 1s ease;
  overflow: hidden;
  font-family: 'Arial', sans-serif;
}

/* カード */
.weather-card {
  background: rgba(255,255,255,0.2);
  backdrop-filter: blur(6px);
  border-radius: 20px;
  padding: 30px;
  text-align: center;
  color: #fff;
  width: 300px;
  box-shadow: 0 8px 20px rgba(0,0,0,0.3);
}

.weather-card h1 {
  margin: 0;
  font-size: 24px;
}
.temp {
  font-size: 48px;
  font-weight: bold;
  margin: 10px 0;
}
.icon i {
  font-size: 50px;
}

/* ==== 背景アニメーション ==== */

/* 晴れ */
.sunny {
  background: linear-gradient(to top, #fac96d, #e7831f, #ffffff);
}

/* 曇り */
.cloudy {
  background: linear-gradient(to top, #757f9a, #d7dde8);
}

/* 雨 */
.rainy {
  background: linear-gradient(to top, #373B44, #1d0bd8);
}

/* 雪 */
.snowy {
  background: linear-gradient(to top, #c4ebf0, #a5c0e8);
}

/* 雷 */
.stormy {
  background: linear-gradient(to top, #232526, #6e720f);
  animation: stormAnim 3s infinite linear;
}
@keyframes stormAnim {
  0%, 90%, 100% { filter: brightness(1); }
  95% { filter: brightness(3); }
}
</style>
