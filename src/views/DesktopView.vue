<template>
 <div class="fixed inset-0 w-full h-screen flex items-center justify-center p-4 lg:p-6 font-sans overflow-hidden" style="user-select: none;">

    <video
      autoplay
      muted
      loop
      playsinline
      class="absolute inset-0 w-full h-full object-cover z-0"
    >
      <source src="../assets/bg-video.mp4" type="video/mp4" />
      Your browser does not support the video tag.
    </video>

    <div class="absolute inset-0 bg-slate-950/75 z-0"></div>

    <div class="relative z-10 w-full max-w-xl md:max-w-2xl lg:max-w-3xl xl:max-w-4xl 2xl:max-w-5xl h-[92vh] max-h-[900px] bg-slate-900/85 backdrop-blur-xl border border-white/10 shadow-2xl rounded-3xl flex flex-col p-4 lg:p-5 xl:p-6 text-white overflow-hidden">


      <div class="flex gap-2 mb-3 lg:mb-4 shrink-0">
        <div class="relative flex-1">
          <input
            type="text"
            v-model="searchQuery"
            @keyup.enter="handleSearch"
            placeholder="Search City..."
            class="w-full bg-white/10 placeholder-slate-300 text-sm lg:text-base py-2.5 lg:py-3 px-4 pl-10 rounded-xl outline-none border border-white/20 focus:bg-white/20 transition"
          />
          <Search class="absolute left-3 top-3 lg:top-3.5 w-4 h-4 lg:w-5 lg:h-5 text-slate-300" />
        </div>
        <button
          @click="fetchCurrentLocationWeather"
          class="bg-sky-500/30 hover:bg-sky-500/50 border border-sky-400/30 px-3 lg:px-4 rounded-xl transition flex items-center justify-center cursor-pointer shrink-0"
          title="Detect Current Location"
        >
          <Navigation class="w-4 h-4 lg:w-5 lg:h-5 text-sky-300" />
        </button>
      </div>

    
      <div v-if="!weatherData && !loading && !errorMessage" class="flex-1 flex flex-col items-center justify-center text-center">
        <div class="w-16 h-16 lg:w-20 lg:h-20 bg-white/10 rounded-full flex items-center justify-center mb-3 border border-white/10">
          <CloudSun class="w-8 h-8 lg:w-10 lg:h-10 text-sky-300" />
        </div>
        <h2 class="text-base lg:text-lg font-bold mb-1">Search City</h2>
        <p class="text-xs lg:text-sm text-slate-300 max-w-[220px]">Find out weather & forecast conditions</p>
      </div>

      
      <div v-if="loading" class="flex-1 flex items-center justify-center">
        <p class="text-sky-400 animate-pulse text-sm lg:text-base font-medium">Loading weather...</p>
      </div>

    
      <div v-if="errorMessage && !loading" class="flex-1 flex flex-col items-center justify-center text-center">
        <div class="w-14 h-14 lg:w-16 lg:h-16 bg-red-500/20 rounded-full flex items-center justify-center mb-3 border border-red-500/30">
          <MapPin class="w-7 h-7 lg:w-8 lg:h-8 text-red-400" />
        </div>
        <h2 class="text-base lg:text-lg font-bold text-red-400 mb-1">City Not Found</h2>
        <p class="text-xs lg:text-sm text-slate-300 max-w-[260px]">{{ errorMessage }}</p>
      </div>

     
      <div v-if="weatherData && !loading" class="flex-1 overflow-y-auto space-y-3 lg:space-y-4 pr-1 custom-scrollbar">

      
        <div class="flex justify-between items-center">
          <div class="flex items-center gap-1.5">
            <MapPin class="w-4 h-4 lg:w-5 lg:h-5 text-sky-400" />
            <h2 class="text-sm lg:text-lg font-bold truncate max-w-[60vw]">{{ weatherData.cityName }}</h2>
          </div>
          <span class="text-xs lg:text-sm text-slate-300">{{ selectedDateText }}</span>
        </div>

  
        <div class="bg-gradient-to-br from-sky-500/20 to-white/5 p-4 lg:p-6 rounded-2xl border border-white/10 shadow-lg">
          <div class="flex items-center justify-between">
            <div>
              <h1 class="text-5xl lg:text-6xl xl:text-7xl font-extrabold leading-none mb-2 lg:mb-3">{{ selectedTemp }}°C</h1>
              <span class="inline-block text-xs lg:text-sm font-semibold text-sky-200 bg-sky-400/20 border border-sky-300/30 px-3 py-1 rounded-full tracking-wide">
                {{ selectedCondition }}
              </span>
              <p class="text-xs lg:text-sm text-slate-300 mt-2">Feels like {{ selectedFeelsLike }}°C</p>
            </div>
            <img
              :src="getWeatherIcon(selectedCode, isNightTime)"
              alt="Weather Icon"
              class="w-24 h-24 lg:w-32 lg:h-32 xl:w-36 xl:h-36 object-contain filter brightness-125 drop-shadow-lg"
            />
          </div>

          
          <div class="grid grid-cols-2 gap-2 lg:gap-3 mt-4 lg:mt-5">
            <div class="flex items-center gap-2 bg-slate-950/30 border border-white/10 rounded-xl px-3 py-2 lg:py-2.5">
              <CloudRain class="w-4 h-4 lg:w-5 lg:h-5 text-sky-300 shrink-0" />
              <div class="leading-tight">
                <p class="text-[10px] lg:text-xs text-slate-400">Precipitation</p>
                <p class="text-xs lg:text-sm font-bold">{{ selectedRainChance }}%</p>
              </div>
            </div>
            <div class="flex items-center gap-2 bg-slate-950/30 border border-white/10 rounded-xl px-3 py-2 lg:py-2.5">
              <ThermometerSnowflake class="w-4 h-4 lg:w-5 lg:h-5 text-blue-300 shrink-0" />
              <div class="leading-tight">
                <p class="text-[10px] lg:text-xs text-slate-400">Cold Level</p>
                <p class="text-xs lg:text-sm font-bold">{{ selectedColdLevel }}</p>
              </div>
            </div>
          </div>
        </div>

       
        <div class="grid grid-cols-2 md:grid-cols-4 gap-2 lg:gap-3">
          <div
            v-for="item in detailItems"
            :key="item.label"
            class="bg-white/5 p-2.5 lg:p-3.5 rounded-xl border border-white/10 flex items-center gap-2.5"
          >
            <component :is="item.icon" class="w-4 h-4 lg:w-5 lg:h-5 text-sky-400 shrink-0" />
            <div class="leading-tight min-w-0">
              <p class="text-[10px] lg:text-xs text-slate-400">{{ item.label }}</p>
              <p class="text-xs lg:text-sm font-bold truncate">{{ item.value }}</p>
            </div>
          </div>
        </div>

        
        <div>
          <h3 class="text-xs lg:text-sm font-semibold text-slate-300 mb-1.5">Hourly Forecast</h3>
          <swiper
            :slides-per-view="4"
            :space-between="8"
            :breakpoints="{
              768: { slidesPerView: 6, spaceBetween: 8 },
              1024: { slidesPerView: 8, spaceBetween: 10 },
              1280: { slidesPerView: 9, spaceBetween: 10 }
            }"
            class="w-full pb-1 cursor-pointer"
          >
            <swiper-slide
              v-for="(hour, idx) in activeHourlyList"
              :key="idx"
            >
              <div class="bg-white/5 p-2 lg:p-2.5 rounded-xl text-center border border-white/10 flex flex-col items-center">
                <p class="text-[10px] lg:text-xs text-slate-300 mb-0.5">{{ hour.timeStr }}</p>
                <img :src="getWeatherIcon(hour.code, hour.isNight)" alt="Icon" class="w-7 h-7 lg:w-9 lg:h-9 object-contain filter brightness-125 my-0.5" />
                <p class="text-xs lg:text-sm font-bold">{{ hour.temp }}°C</p>
              </div>
            </swiper-slide>
          </swiper>
        </div>


        <div class="pb-5">
          <h3 class="text-xs lg:text-sm font-semibold text-slate-300 mb-1.5">16-Day Forecast</h3>
          <swiper
            :slides-per-view="3.2"
            :space-between="8"
            :breakpoints="{
              768: { slidesPerView: 4.2, spaceBetween: 8 },
              1024: { slidesPerView: 5.2, spaceBetween: 10 },
              1280: { slidesPerView: 6.2, spaceBetween: 10 }
            }"
            class="w-full pb-1 cursor-grab active:cursor-grabbing"
          >
            <swiper-slide
              v-for="(day, index) in weatherData.daily"
              :key="index"
            >
              <div
                @click="selectDayForecast(day)"
                :class="[
                  'p-2 lg:p-2.5 rounded-xl text-center border transition flex flex-col items-center cursor-pointer',
                  selectedFullDate === day.fullDate ? 'bg-sky-500/30 border-sky-400 shadow-md' : 'bg-white/5 border-white/10 hover:bg-white/10'
                ]"
              >
                <p class="text-[10px] lg:text-xs text-slate-300 mb-0.5">{{ day.dayDisplay }}</p>
                <img
                  :src="getWeatherIcon(day.code, false)"
                  alt="Icon"
                  class="w-7 h-7 lg:w-9 lg:h-9 object-contain filter brightness-125 my-0.5"
                />
                <p class="text-xs lg:text-sm font-bold">{{ day.temp }}°C</p>
                <span v-if="day.temp < 20" class="text-[10px] lg:text-xs text-blue-300 mt-0.5">❄️ Cold</span>
                <span v-else class="text-[10px] lg:text-xs text-sky-300 mt-0.5">{{ day.rainChance }}% rain</span>
              </div>
            </swiper-slide>
          </swiper>
        </div>

      </div>

    </div>
  </div>
</template>






<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import {
  Search, MapPin, Droplets, Wind, CloudSun, Navigation, CloudRain, ThermometerSnowflake,
  Thermometer, Waves, Gauge, Eye, ArrowUpDown, Umbrella
} from 'lucide-vue-next';
import { Swiper, SwiperSlide } from 'swiper/vue';
import 'swiper/css';

const WEATHER_API = 'https://api.open-meteo.com/v1/forecast';

const searchQuery = ref('');
const loading = ref(false);
const errorMessage = ref('');
const weatherData = ref<any>(null);

const selectedTemp = ref(0);
const selectedCondition = ref('');
const selectedCode = ref(0);
const selectedHumidity = ref(0);
const selectedWindSpeed = ref(0);
const selectedFeelsLike = ref(0);
const selectedRainChance = ref(0);
const selectedColdLevel = ref('');
const selectedDateText = ref('');
const selectedFullDate = ref('');
const isNightTime = ref(false);
const activeHourlyList = ref<any[]>([]);


const selectedDewPoint = ref(0);
const selectedPressure = ref(0);
const selectedVisibility = ref(0);
const selectedRainAmount = ref(0);
const selectedForecastRange = ref('');

const detailItems = computed(() => [
  { label: 'Humidity', value: `${selectedHumidity.value}%`, icon: Droplets },
  { label: 'Wind Speed', value: `${selectedWindSpeed.value} m/s`, icon: Wind },
  { label: 'Feels Like', value: `${selectedFeelsLike.value}°C`, icon: Thermometer },
  { label: 'Dew Point', value: `${selectedDewPoint.value}°C`, icon: Waves },
  { label: 'Pressure', value: `${selectedPressure.value} hPa`, icon: Gauge },
  { label: 'Visibility', value: `${selectedVisibility.value} km`, icon: Eye },
  { label: 'Forecast', value: selectedForecastRange.value, icon: ArrowUpDown },
  { label: 'Rain Amount', value: `${selectedRainAmount.value} mm`, icon: Umbrella },
]);

const getWeatherIcon = (code: number, night: boolean) => {
  if (code === 0) return night ? 'https://openweathermap.org/img/wn/01n@2x.png' : 'https://openweathermap.org/img/wn/01d@2x.png';
  if (code <= 3) return night ? 'https://openweathermap.org/img/wn/02n@2x.png' : 'https://openweathermap.org/img/wn/02d@2x.png';
  if (code === 45 || code === 48) return 'https://openweathermap.org/img/wn/50d@2x.png';
  if (code <= 67 || (code >= 80 && code <= 82)) return night ? 'https://openweathermap.org/img/wn/10n@2x.png' : 'https://openweathermap.org/img/wn/10d@2x.png';
  if (code <= 77 || code === 85 || code === 86) return 'https://openweathermap.org/img/wn/13d@2x.png';
  if (code >= 95) return 'https://openweathermap.org/img/wn/11d@2x.png';
  return night ? 'https://openweathermap.org/img/wn/03n@2x.png' : 'https://openweathermap.org/img/wn/03d@2x.png';
};

const getWeatherCondition = (code: number) => {
  if (code === 0) return 'Clear Sky';
  if (code <= 3) return 'Partly Cloudy';
  if (code === 45 || code === 48) return 'Foggy';
  if (code <= 67) return 'Precipitation';
  if (code >= 95) return 'Thunderstorm';
  return 'Cloudy';
};

const getColdLevelText = (temp: number) => {
  if (temp < 15) return 'Severe Cold';
  if (temp < 20) return 'Moderate Cold';
  return 'Normal';
};

const avg = (arr: number[], digits = 0) => {
  if (!arr.length) return 0;
  const v = arr.reduce((s, n) => s + n, 0) / arr.length;
  return Number(v.toFixed(digits));
};

const buildHourly = (h: any) =>
  h.time.map((t: string, i: number) => {
    const hr = Number(t.slice(11, 13));
    return {
      date: t.slice(0, 10),
      timeStr: `${hr % 12 || 12} ${hr >= 12 ? 'PM' : 'AM'}`,
      temp: Math.round(h.temperature_2m[i]),
      code: h.weather_code[i],
      isNight: !h.is_day[i],
      humidity: h.relative_humidity_2m[i],
      windSpeed: h.wind_speed_10m[i],
      feelsLike: Math.round(h.apparent_temperature[i]),
      rainChance: h.precipitation_probability[i] ?? 0,
      dewPoint: h.dew_point_2m[i],
      pressure: h.pressure_msl[i],
      visibility: h.visibility[i],
    };
  });

const loadWeather = async (lat: number, lon: number, cityName: string) => {
  const url =
    `${WEATHER_API}?latitude=${lat}&longitude=${lon}` +
    `&current=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m,is_day,dew_point_2m,pressure_msl,visibility,precipitation` +
    `&hourly=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m,precipitation_probability,is_day,dew_point_2m,pressure_msl,visibility` +
    `&daily=weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,precipitation_probability_max,wind_speed_10m_max,precipitation_sum` +
    `&forecast_days=16&timezone=auto&wind_speed_unit=ms`;

  const data = await (await fetch(url)).json();
  const allHourly = buildHourly(data.hourly);

  const dailyList = data.daily.time.map((date: string, i: number) => {
    const hourlyData = allHourly.filter((h: any) => h.date === date);
    const temp = Math.round(data.daily.temperature_2m_max[i]);
    const minTemp = Math.round(data.daily.temperature_2m_min[i]);
    const code = data.daily.weather_code[i];

    return {
      dayDisplay: new Date(date + 'T00:00:00').toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' }),
      fullDate: date,
      temp,
      code,
      condition: getWeatherCondition(code),
      humidity: avg(hourlyData.map((h: any) => h.humidity)),
      windSpeed: data.daily.wind_speed_10m_max[i],
      feelsLike: Math.round(data.daily.apparent_temperature_max[i]),
      rainChance: data.daily.precipitation_probability_max[i] ?? 0,
      coldLevel: getColdLevelText(temp),
      dewPoint: avg(hourlyData.map((h: any) => h.dewPoint)),
      pressure: avg(hourlyData.map((h: any) => h.pressure)),
      visibility: Number((avg(hourlyData.map((h: any) => h.visibility)) / 1000).toFixed(1)),
      rainAmount: data.daily.precipitation_sum[i] ?? 0,
      forecastRange: `${minTemp}° / ${temp}°`,
      hourlyData,
    };
  });

  weatherData.value = { cityName, daily: dailyList };


  selectDayForecast(dailyList[0]);
  const c = data.current;
  selectedTemp.value = Math.round(c.temperature_2m);
  selectedCondition.value = getWeatherCondition(c.weather_code);
  selectedCode.value = c.weather_code;
  selectedHumidity.value = c.relative_humidity_2m;
  selectedWindSpeed.value = c.wind_speed_10m;
  selectedFeelsLike.value = Math.round(c.apparent_temperature);
  selectedColdLevel.value = getColdLevelText(selectedTemp.value);
  selectedDewPoint.value = Math.round(c.dew_point_2m);
  selectedPressure.value = Math.round(c.pressure_msl);
  selectedVisibility.value = Number((c.visibility / 1000).toFixed(1));
  selectedRainAmount.value = c.precipitation ?? 0;
  isNightTime.value = !c.is_day;
};

const selectDayForecast = (day: any) => {
  selectedTemp.value = day.temp;
  selectedCondition.value = day.condition;
  selectedCode.value = day.code;
  selectedHumidity.value = day.humidity;
  selectedWindSpeed.value = day.windSpeed;
  selectedFeelsLike.value = day.feelsLike;
  selectedRainChance.value = day.rainChance;
  selectedColdLevel.value = day.coldLevel;
  selectedDateText.value = day.dayDisplay;
  selectedFullDate.value = day.fullDate;
  selectedDewPoint.value = day.dewPoint;
  selectedPressure.value = day.pressure;
  selectedVisibility.value = day.visibility;
  selectedRainAmount.value = day.rainAmount;
  selectedForecastRange.value = day.forecastRange;
  isNightTime.value = false;
  activeHourlyList.value = day.hourlyData;
};

const fetchWeather = async (city: string) => {
  if (!city.trim()) return;
  loading.value = true;
  errorMessage.value = '';
  weatherData.value = null;
  try {
    const geo = await (
      await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}&count=1`)
    ).json();

    if (!geo.results?.length) {
      errorMessage.value = `"${city}" namer kono city paoa jayni. Sothik banan diye abar chesta korun!`;
      return;
    }
    const { latitude, longitude, name, country } = geo.results[0];
    await loadWeather(latitude, longitude, `${name}, ${country || ''}`);
  } catch (err) {
    console.error(err);
    errorMessage.value = 'Unable to load data. Please check your internet connection and try again.';
  } finally {
    loading.value = false;
  }
};

const fetchCurrentLocationWeather = () => {
  if (!navigator.geolocation) return fetchWeather('Tangail');
  loading.value = true;
  errorMessage.value = '';
  navigator.geolocation.getCurrentPosition(
    async ({ coords }) => {
      try {
        await loadWeather(coords.latitude, coords.longitude, 'My Current Location');
      } catch {
        fetchWeather('Tangail');
      } finally {
        loading.value = false;
      }
    },
    () => fetchWeather('Tangail')
  );
};

const handleSearch = () => {
  if (searchQuery.value.trim()) fetchWeather(searchQuery.value);
};

onMounted(() => fetchWeather('Tangail'));
</script>