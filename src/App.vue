<template>
  <div id="main" :class="isDay ? 'day' : 'night'">
    <div class="container my-5" style="max-width: 440px; min-width: 460px;">
      <h1 class="title text-center my-3">Weather  Vue</h1>
      <form class="search-location" v-on:submit.prevent="getWeather">
        <div class="input-container">
          <input
            type="text"
            class="form-control text-muted form-rounded p-4 shadow-sm"
            placeholder="Digite a cidade ou estado"
            v-model="citySearch"
            @input="getCities"
          />
          <div v-if="filteredCities.length > 0 && citySearch.length > 0">
            <ul>
              <li v-for="city in filteredCities" :key="city.id" @click="selectCity(city)">{{ city }}</li>
            </ul>
          </div>
          <img @click="getWeather" class="search" src="./assets/img/search.svg" alt="img search" />
        </div>
      </form>
      
      <p class="text-center my-3" v-if="cityFound">Cidade não encontrada!</p>
      <div
        class="card rounded my-3 shadow-lg back-card overflow-hidden"
        v-if="visible"
      >
     
        <!-- container de animação do clima  -->
        <div>
          <div icon="sunny" v-if="clearSky" data-label="Sunny">
            <span class="sun"></span>
          </div>

          <div icon="snowy" v-if="snowy" data-label="Chilly">
            <ul>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
            </ul>
          </div>

          <div icon="stormy" v-if="stormy" data-label="Soggy">
            <span class="cloud"></span>
            <ul>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
            </ul>
          </div>
          <div icon="cloudy" v-if="cloudy" data-label="Perfect">
            <span class="cloud"></span>
            <span class="cloud"></span>
          </div>
          <div icon="nightmoon" v-if="clearNight" data-label="Cool!">
            <span class="moon"></span>
            <span class="meteor"></span>
          </div>
        </div>
       
        <div class="card-top text-center" style="margin-bottom: 15rem">
          <div class="city-name my-3">
            <p>{{ weather.cityName }} -  {{ weather.country }}</p>
          </div>
          <div class="date my-5"> 
            {{ dateBuilder() }}
          </div>
        </div>

        <div class="card-body">
          <div class="card-mid">
            <div class="row">
              <div class="col-12  text-center temp">
                <span>{{ weather.temperature }}&deg;C</span>
                <p class="my-4">{{ weather.description }}</p>
              </div>
            </div>
            <div class="row">
              <div class="col d-flex justify-content-between px-6 mx-5">
                <span class="py-2">
                  <img src="./assets/img/down.svg" alt="img down" />
                  {{ weather.lowTemp }}&deg;C 
                  <br>
                  <br>
                  <span><strong>Temp.Mínima</strong></span>
                </span>
                <span class="py-2">
                  <img src="./assets/img/up.svg" alt="img up" />
                  {{ weather.highTemp }}&deg;C
                  <br>
                  <br>
                  <span><strong>Temp.Máxima</strong></span>
                </span>
              </div>
            </div>
          </div>

          <div class="card-bottom px-6 py-4 row">
            <div class="col text-center">
              <p>{{ weather.feelsLike }}&deg;C</p>
              <span>Sensação Térmica</span>
            </div>
            <div class="col text-center">
              <p>{{ weather.humidity }}%</p>
              <span>Umidade</span>
            </div>
          </div>
          
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { debounce } from 'lodash';
export default {
  data() {
    return {
      cityFound: false,
      visible: false,
      stormy: false,
      cloudy: false,
      clearSky: false,
      clearNight: false,
      snowy: false,
      isDay: true,
      citySearch: "",
      weather: {
        cityName: "São Paulo",
        country: "BR",
        temperature: 25,
        description: "céu limpo",
        lowTemp: "20",
        highTemp: "25",
        feelsLike: "23",
        humidity: "77",
      },
      filteredCities: [],
    };
  },
  methods: {
    getCities: debounce(function()  {
      const key = process.env.VUE_APP_API_KEY
      
       if (this.citySearch === '') {
        this.filteredCities = [];
        return;
      } 

      axios.get(`https://api.openweathermap.org/data/2.5/find?q=${this.citySearch}&type=like&sort=population&cnt=10&appid=${key}`)
        .then(response => {
          const cityNames = response.data.list.map(city => city.name);
          this.filteredCities = cityNames;
        })
        .catch(error => {
          console.log(error);
        });
    },500),
    selectCity(city) {
      this.citySearch = city;
      this.filteredCities = [];
      this.getWeather();
    },
    getWeather: async function () {
      const key = process.env.VUE_APP_API_KEY
      const baseURL = `https://api.openweathermap.org/data/2.5/weather?q=${this.citySearch}&appid=${key}&lang=pt_br&units=metric`;
      
      //buscar o clima
      try {
        const response = await fetch(baseURL);
        const data = await response.json();
        this.citySearch = "";
        this.weather.cityName = data.name;
        this.weather.country = data.sys.country;
        this.weather.temperature = Math.round(data.main.temp);
        this.weather.description = data.weather[0].description;
        this.weather.lowTemp = Math.round(data.main.temp_min);
        this.weather.highTemp = Math.round(data.main.temp_max);
        this.weather.feelsLike = Math.round(data.main.feels_like);
        this.weather.humidity = Math.round(data.main.humidity);

        const timeOfDay = data.weather[0].icon;

        //verificar se é dia
        if (timeOfDay.includes("n")) {
          this.isDay = false;
        } else {
          this.isDay = true;
        }

        const mainWeather = data.weather[0].main;
        //verificar animações de determinado clima
        if (mainWeather.includes("Clouds")) {
          this.stormy = false;
          this.cloudy = true;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clouds")) {
          this.stormy = false;
          this.cloudy = true;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (
          mainWeather.includes("Thunderstorm") ||
          mainWeather.includes("Rain")
        ) {
          this.stormy = true;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clear") && this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = true;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clear") && !this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = true;
          this.snowy = false;
        }
        if (mainWeather.includes("Clouds") && !this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = true;
          this.snowy = false;
        }
        if (mainWeather.includes("Snow")) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = true;
        }

        this.visible = true;
        this.cityFound = false;
      } catch (error) {
        console.log(error);
        this.cityFound = true;
        this.visible = false;
      }
    },
     dateBuilder() {
      let d = new Date();
      let options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', hour: 'numeric', minute: 'numeric' };
      let date = d.toLocaleDateString('pt-BR', options);
      return date;
    },    
  },
};
</script>

<style scoped>
@import "./assets/css/custom.css";
@import "./assets/css/animation.css";
</style>
