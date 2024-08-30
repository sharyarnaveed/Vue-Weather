<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";

const api_key = import.meta.env.VITE_VUE_API_KEY;
const searchval = ref("");
const searchresult = ref(null);
const thetempc = ref("0");
const windspeed = ref("0");
const humidity = ref("0");
const imgurl = ref('');
const weatherdays = ref([]);
const location = ref('');
const date = ref('');
const storelocal = ref('');
const storelocal2 = ref(null);

const searchIconRef = ref(null);
const searchDivRef = ref(null);

onMounted(() => {
  const today = new Date();
  const dd = String(today.getDate()).padStart(2, '0');
  const mm = String(today.getMonth() + 1).padStart(2, '0');
  const yyyy = today.getFullYear();
  date.value = mm + '/' + dd + '/' + yyyy;

  // Get data from localStorage
  const storedData = localStorage.getItem('storelocal');
  if (storedData) {
    storelocal2.value = JSON.parse(storedData);
    if (storelocal2.value.length > 0) {
      location.value = storelocal2.value[0].location;
      thetempc.value = storelocal2.value[0].temp;
      windspeed.value = storelocal2.value[0].windspeed;
      humidity.value = storelocal2.value[0].humidity;
      imgurl.value = storelocal2.value[0].imgurl;
    }
  }
});

const callapi = async () => {
  try {
    const response = await axios.get(
      `https://api.weatherapi.com/v1/forecast.json?key=${api_key}&q=${searchval.value}&days=4&aqi=no&alerts=yes`
    );
    searchresult.value = response.data;
    location.value = searchresult.value.location.name;
    thetempc.value = searchresult.value.current.temp_c;
    windspeed.value = searchresult.value.current.wind_kph;
    humidity.value = searchresult.value.current.humidity;
    imgurl.value = searchresult.value.current.condition.icon;
    weatherdays.value = searchresult.value.forecast.forecastday;

    // Storing data in localStorage
    storelocal.value = [
      {
        location: location.value,
        date: date.value,
        temp: thetempc.value,
        humidity: humidity.value,
        windspeed: windspeed.value,
        imgurl: imgurl.value,
      }
    ];

    localStorage.setItem('storelocal', JSON.stringify(storelocal.value));
  } catch (error) {
    console.log(error);
  }
};

const handlesearch = (e) => {
  if (e.key === "Enter") {
    callapi();
  }
};

const opensearch = () => {
  if (searchIconRef.value) {
    searchIconRef.value.style.display = "none";
  }
  if (searchDivRef.value) {
    searchDivRef.value.style.display = "flex";
  }
};

const close_search = () => {
  if (searchIconRef.value) {
    searchIconRef.value.style.display = "flex";
  }
  if (searchDivRef.value) {
    searchDivRef.value.style.display = "none";
  }
};
</script>

<template>
  <main class="weather_container">
    <section class="main_weather">
      <div class="left_side">
        <div class="top_date">
          <h3 id="city_name">{{ location }}</h3>
          <h3 id="date_show">{{ date }}</h3>
        </div>
        <div class="bottom_in_main">
          <div class="temperature">
            <div id="image_temp">
              <img id="weather_image" :src="imgurl" alt="Weather Icon" />
            </div>
            <h1 id="temperature">{{ thetempc }} °C</h1>
            <h2 id="condition">{{ searchresult?.value?.current?.condition?.text || '' }}</h2>
          </div>
          <div class="humidity">
            <h3 id="wind_speed">
              Wind Speed
              <br />
              {{ windspeed }} kph
            </h3>
            <h3 id="humidity">
              Humidity
              <br />
              {{ humidity }}%
            </h3>
          </div>
        </div>
        <div class="other_weather" id="days_weather">
          <div
            v-for="(theday, index) in weatherdays"
            :key="index"
            class="weather_cards"
          >
            <div class="weather_card_day" style="color: white;">{{ theday.date }}</div>
            <div class="card_content">
              <img :src="theday.day.condition.icon" alt="Weather Icon" />
            </div>
            <div class="weather_card_temp">
              <h2>{{ theday.day.avgtemp_c }} °C</h2>
            </div>
          </div>
        </div>
      </div>

      <aside class="right_side">
        <div ref="searchIconRef" class="search_icon">
          <button @click="opensearch" class="icon">
            <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="50" height="45" viewBox="0 0 50 50">
              <path d="M 21 3 C 11.622998 3 4 10.623005 4 20 C 4 29.376995 11.622998 37 21 37 C 24.712383 37 28.139151 35.791079 30.9375 33.765625 L 44.085938 46.914062 L 46.914062 44.085938 L 33.886719 31.058594 C 36.443536 28.083 38 24.223631 38 20 C 38 10.623005 30.377002 3 21 3 z M 21 5 C 29.296122 5 36 11.703883 36 20 C 36 28.296117 29.296122 35 21 35 C 12.703878 35 6 28.296117 6 20 C 6 11.703883 12.703878 5 21 5 z"></path>
            </svg>
          </button>
        </div>

        <div ref="searchDivRef" class="search">
          <div class="closeicon" @click="close_search">
            <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="30" height="30" viewBox="0 0 50 50">
              <path d="M 7.71875 6.28125 L 6.28125 7.71875 L 23.5625 25 L 6.28125 42.28125 L 7.71875 43.71875 L 25 26.4375 L 42.28125 43.71875 L 43.71875 42.28125 L 26.4375 25 L 43.71875 7.71875 L 42.28125 6.28125 L 25 23.5625 Z"></path>
            </svg>
          </div>
          <label for="search">Search:</label>
          <input
            type="search"
            id="search"
            @keydown="handlesearch"
            v-model="searchval"
          />

          <div class="search_icon_svg_inner" @click="callapi">
            <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="30" height="40" viewBox="0 0 50 50">
              <path d="M 21 3 C 11.622998 3 4 10.623005 4 20 C 4 29.376995 11.622998 37 21 37 C 24.712383 37 28.139151 35.791079 30.9375 33.765625 L 44.085938 46.914062 L 46.914062 44.085938 L 33.886719 31.058594 C 36.443536 28.083 38 24.223631 38 20 C 38 10.623005 30.377002 3 21 3 z M 21 5 C 29.296122 5 36 11.703883 36 20 C 36 28.296117 29.296122 35 21 35 C 12.703878 35 6 28.296117 6 20 C 6 11.703883 12.703878 5 21 5 z"></path>
            </svg>
          </div>
        </div>

      
      </aside>
    </section>
  </main>
</template>
