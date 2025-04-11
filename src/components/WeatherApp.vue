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
  background: linear-gradient(to bottom right, #00b4db, #0083b0);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
}

.container {
  background: rgba(255, 255, 255, 0.95);
  padding: 3rem;
  border-radius: 25px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
  width: 100%;
  max-width: 1000px;
  margin: 40px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.search-box {
  display: flex;
  gap: 15px;
  margin-bottom: 3rem;
}

.search-box input {
  flex: 1;
  padding: 15px 20px;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  font-size: 18px;
  outline: none;
  transition: border-color 0.3s;
}

.search-box input:focus {
  border-color: #0083b0;
}

.search-box button {
  padding: 15px 30px;
  background: #0083b0;
  border: none;
  border-radius: 12px;
  color: white;
  cursor: pointer;
  transition: all 0.3s;
  min-width: 60px;
  font-size: 18px;
}

.search-box button:hover {
  background: #00b4db;
  transform: translateY(-2px);
}

.weather-info {
  text-align: center;
  padding: 20px;
}

.location h2 {
  margin-bottom: 1.5rem;
  color: #333;
  font-size: clamp(1.8rem, 3vw, 3rem);
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}

.temperature {
  margin: 3rem 0;
  padding: 20px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 20px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
}

.temperature h1 {
  font-size: clamp(4rem, 8vw, 8rem);
  margin: 0;
  color: #0083b0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.temperature img {
  width: clamp(100px, 25vw, 200px);
  margin: 1.5rem 0;
}

.temperature p {
  font-size: clamp(1.2rem, 2vw, 2rem);
  color: #555;
  margin-top: 1rem;
}

.details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 25px;
  margin-top: 3rem;
  padding-top: 3rem;
  border-top: 2px solid #eee;
}

.detail {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 25px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 15px;
  transition: all 0.3s;
  border: 1px solid rgba(0, 0, 0, 0.05);
}

.detail:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
  background: rgba(255, 255, 255, 0.9);
}

.detail i {
  font-size: clamp(2rem, 4vw, 3rem);
  color: #0083b0;
}

.detail div p:first-child {
  font-size: clamp(1rem, 1.8vw, 1.2rem);
  color: #666;
  margin-bottom: 8px;
}

.detail div p:last-child {
  font-size: clamp(1.2rem, 2.5vw, 2rem);
  color: #333;
  font-weight: 600;
}

.error {
  color: #dc3545;
  text-align: center;
  margin-top: 2rem;
  font-size: clamp(1rem, 1.8vw, 1.2rem);
  padding: 15px;
  background: rgba(220, 53, 69, 0.1);
  border-radius: 12px;
  border: 1px solid rgba(220, 53, 69, 0.2);
}

.debug-info {
  font-size: 0.9rem;
  margin-top: 1rem;
  color: #666;
  padding: 10px;
  background: rgba(0, 0, 0, 0.03);
  border-radius: 8px;
}

.loading {
  text-align: center;
  margin-top: 3rem;
  padding: 20px;
}

.loading i {
  font-size: 3rem;
  color: #0083b0;
  animation: spin 1s linear infinite;
}

.loading p {
  margin-top: 1rem;
  font-size: 1.2rem;
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
    max-width: 700px;
    padding: 2rem;
  }

  .details {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 769px) and (max-width: 1024px) {
  .container {
    max-width: 900px;
    padding: 2.5rem;
  }

  .details {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (min-width: 1025px) {
  .container {
    max-width: 1000px;
    padding: 3rem;
  }

  .details {
    grid-template-columns: repeat(3, 1fr);
  }
}
</style>
