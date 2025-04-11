<template>
  <div class="weather-app">
    <div class="container">
      <div class="search-box">
        <input
          type="text"
          placeholder="Digite o nome da cidade..."
          v-model="city"
          @keyup.enter="getWeather"
        />
        <button @click="getWeather">
          <i class="mdi mdi-magnify"></i>
        </button>
      </div>

      <div class="weather-info" v-if="weatherData">
        <div class="location">
          <h2>
            {{ weatherData.location.name }}, {{ weatherData.location.country }}
          </h2>
        </div>

        <div class="temperature">
          <h1>{{ Math.round(weatherData.current.temp_c) }}°C</h1>
          <img
            :src="weatherData.current.condition.icon"
            :alt="weatherData.current.condition.text"
          />
          <p>{{ weatherData.current.condition.text }}</p>
        </div>

        <div class="details">
          <div class="detail">
            <i class="mdi mdi-water"></i>
            <div>
              <p>Umidade</p>
              <p>{{ weatherData.current.humidity }}%</p>
            </div>
          </div>
          <div class="detail">
            <i class="mdi mdi-weather-windy"></i>
            <div>
              <p>Vento</p>
              <p>{{ Math.round(weatherData.current.wind_kph) }} km/h</p>
            </div>
          </div>
        </div>
      </div>

      <div class="error" v-if="errorMessage">
        {{ errorMessage }}
        <div v-if="debugInfo" class="debug-info">
          <p>Detalhes do erro: {{ debugInfo }}</p>
        </div>
      </div>

      <div class="loading" v-if="loading">
        <i class="mdi mdi-loading mdi-spin"></i>
        <p>Carregando...</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";

interface WeatherCondition {
  text: string;
  icon: string;
}

interface CurrentWeather {
  temp_c: number;
  humidity: number;
  wind_kph: number;
  condition: WeatherCondition;
}

interface Location {
  name: string;
  country: string;
}

interface WeatherData {
  location: Location;
  current: CurrentWeather;
}

const city = ref("");
const weatherData = ref<WeatherData | null>(null);
const errorMessage = ref("");
const loading = ref(false);
const debugInfo = ref("");

const getWeather = async () => {
  if (!city.value) {
    errorMessage.value = "Por favor, digite o nome de uma cidade";
    return;
  }

  try {
    loading.value = true;
    errorMessage.value = "";
    debugInfo.value = "";

    const normalizedCity = city.value.trim().toLowerCase();
    const url = `https://api.weatherapi.com/v1/current.json?key=aef92fe51e95401a992234225251004&q=${encodeURIComponent(
      normalizedCity
    )}&lang=pt`;
    
    const response = await axios.get(url);
    weatherData.value = response.data;
  } catch (error: any) {
    if (error.response?.status === 404) {
      errorMessage.value = "Cidade não encontrada. Por favor, verifique o nome e tente novamente.";
    } else if (error.response?.status === 401) {
      errorMessage.value = "Erro de autenticação. Por favor, tente novamente mais tarde.";
    } else {
      errorMessage.value = "Ocorreu um erro ao buscar os dados. Por favor, tente novamente mais tarde.";
    }
    weatherData.value = null;
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.weather-app {
  min-height: 100vh;
  background: linear-gradient(to bottom right, #00b4db, #0083b0);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.container {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 400px;
}

.search-box {
  display: flex;
  gap: 10px;
  margin-bottom: 2rem;
}

.search-box input {
  flex: 1;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  outline: none;
}

.search-box button {
  padding: 12px;
  background: #0083b0;
  border: none;
  border-radius: 8px;
  color: white;
  cursor: pointer;
  transition: background 0.3s;
}

.search-box button:hover {
  background: #00b4db;
}

.weather-info {
  text-align: center;
}

.location h2 {
  margin-bottom: 1rem;
  color: #333;
}

.temperature {
  margin: 2rem 0;
}

.temperature h1 {
  font-size: 4rem;
  margin: 0;
  color: #0083b0;
}

.temperature img {
  width: 100px;
  margin: 1rem 0;
}

.details {
  display: flex;
  justify-content: space-around;
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #eee;
}

.detail {
  display: flex;
  align-items: center;
  gap: 10px;
}

.detail i {
  font-size: 2rem;
  color: #0083b0;
}

.error {
  color: #dc3545;
  text-align: center;
  margin-top: 1rem;
}

.debug-info {
  font-size: 0.8rem;
  margin-top: 0.5rem;
  color: #666;
}

.loading {
  text-align: center;
  margin-top: 2rem;
}

.loading i {
  font-size: 2rem;
  color: #0083b0;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>
