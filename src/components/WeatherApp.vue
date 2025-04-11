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
import axios, { AxiosError } from "axios";

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
    
    const response = await axios.get<WeatherData>(url);
    weatherData.value = response.data;
  } catch (err) {
    const error = err as AxiosError;
    let message = "Ocorreu um erro ao buscar os dados. Por favor, tente novamente mais tarde.";
    
    if (error.response?.status === 404) {
      message = "Cidade não encontrada. Por favor, verifique o nome e tente novamente.";
    } else if (error.response?.status === 401) {
      message = "Erro de autenticação. Por favor, tente novamente mais tarde.";
    }
    
    errorMessage.value = message;
    weatherData.value = null;
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.weather-app {
  min-height: 100vh;
  width: 100vw;
  background: linear-gradient(to bottom right, #00b4db, #0083b0);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow-y: auto;
}

.container {
  background: rgba(255, 255, 255, 0.95);
  padding: 2rem;
  border-radius: 20px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
  width: 100%;
  max-width: 500px;
  margin: 40px auto;
  border: 1px solid rgba(255, 255, 255, 0.2);
  position: relative;
  z-index: 1;
}

.search-box {
  display: flex;
  gap: 10px;
  margin-bottom: 2rem;
}

.search-box input {
  flex: 1;
  padding: 12px 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  outline: none;
  transition: border-color 0.3s;
}

.search-box input:focus {
  border-color: #0083b0;
}

.search-box button {
  padding: 12px;
  background: #0083b0;
  border: none;
  border-radius: 8px;
  color: white;
  cursor: pointer;
  transition: background 0.3s;
  min-width: 50px;
}

.search-box button:hover {
  background: #00b4db;
}

.weather-info {
  text-align: center;
  padding: 10px;
}

.location h2 {
  margin-bottom: 1rem;
  color: #333;
  font-size: clamp(1.5rem, 2.5vw, 2rem);
}

.temperature {
  margin: 2rem 0;
  padding: 15px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
}

.temperature h1 {
  font-size: clamp(3rem, 6vw, 5rem);
  margin: 0;
  color: #0083b0;
}

.temperature img {
  width: clamp(80px, 20vw, 120px);
  margin: 1rem 0;
}

.temperature p {
  font-size: clamp(1.2rem, 2vw, 1.5rem);
  color: #555;
  margin-top: 0.5rem;
}

.details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #eee;
}

.detail {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 10px;
  transition: all 0.3s;
}

.detail:hover {
  transform: translateY(-5px);
  background: rgba(255, 255, 255, 0.9);
}

.detail i {
  font-size: clamp(1.5rem, 3vw, 2rem);
  color: #0083b0;
}

.detail div p:first-child {
  font-size: clamp(0.8rem, 1.5vw, 1rem);
  color: #666;
  margin-bottom: 4px;
}

.detail div p:last-child {
  font-size: clamp(1rem, 2vw, 1.2rem);
  color: #333;
  font-weight: 500;
}

.error {
  color: #dc3545;
  text-align: center;
  margin-top: 1rem;
  font-size: clamp(0.9rem, 1.5vw, 1rem);
  padding: 10px;
  background: rgba(220, 53, 69, 0.1);
  border-radius: 8px;
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
  font-size: 2.5rem;
  color: #0083b0;
  animation: spin 1s linear infinite;
}

.loading p {
  margin-top: 1rem;
  font-size: 1.1rem;
  color: #555;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* Media Queries para diferentes tamanhos de tela */
@media (max-width: 480px) {
  .weather-app {
    padding: 20px;
  }

  .container {
    padding: 1.5rem;
    margin: 10px;
  }

  .search-box {
    flex-direction: column;
  }

  .search-box button {
    width: 100%;
  }

  .details {
    grid-template-columns: 1fr;
  }

  .temperature h1 {
    font-size: 3.5rem;
  }
}

@media (min-width: 481px) and (max-width: 768px) {
  .container {
    max-width: 500px;
    padding: 2rem;
  }

  .details {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 769px) and (max-width: 1024px) {
  .container {
    max-width: 500px;
    padding: 2rem;
  }

  .details {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1025px) {
  .container {
    max-width: 500px;
    padding: 2rem;
  }

  .details {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
