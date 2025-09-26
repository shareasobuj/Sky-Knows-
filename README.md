 Sky-Knows
 
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title> Sky Knows </title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(to right, #74ABE2, #5563DE);
      color: white;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
    }
    .forecast-container {
      background: rgba(0,0,0,0.3);
      border-radius: 10px;
      padding: 20px;
      margin-top: 30px;
      width: 350px;
    }
    .forecast-container h1 {
      text-align: center;
      margin-bottom: 20px;
    }
    .day-forecast {
      background: rgba(255, 255, 255, 0.1);
      border-radius: 8px;
      padding: 10px;
      margin-bottom: 15px;
    }
    .day-forecast h2 {
      margin: 0 0 10px;
    }
    .temps {
      display: flex;
      justify-content: space-between;
      margin-bottom: 5px;
    }
    .sky {
      text-transform: capitalize;
    }
    .search-box {
      margin-top: 20px;
      text-align: center;
    }
    .search-box input {
      padding: 5px;
      width: 200px;
      border: none;
      border-radius: 5px;
    }
    .search-box button {
      padding: 5px 10px;
      border: none;
      border-radius: 5px;
      background: #333;
      color: white;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="forecast-container">
    <h1>Sky Knows <span id="city-name">Dhaka</span></h1>
    <div id="forecast-days">
      <!-- forecast cards হবে এখানে -->
    </div>

    <div class="search-box">
      <input type="text" id="city-input" placeholder="Enter city name" />
      <button id="search-btn">Search</button>
    </div>
  </div>

  <script>
    // শহরের নাম থেকে latitude & longitude পেতে geocoding API
    async function geocodeCity(city) {
      const url = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}`;
      const resp = await fetch(url);
      const data = await resp.json();
      if (data && data.results && data.results.length > 0) {
        return {
          latitude: data.results[0].latitude,
          longitude: data.results[0].longitude,
          name: data.results[0].name,
        };
      } else {
        throw new Error("City not found");
      }
    }

    // 3 দিনের forecast আনতে হবে daily API অংশ দিয়ে
    async function fetchForecast(lat, lon) {
      // daily param: temperature_2m_max, temperature_2m_min, weathercode
      // forecast_days=3, timezone auto
      const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=temperature_2m_max,temperature_2m_min,weathercode&forecast_days=3&timezone=auto`;
      const resp = await fetch(url);
      const data = await resp.json();
      return data;
    }

    // weathercode সংখ্যা → বর্ণনায় রূপান্তর
    function weatherCodeToDescription(code) {
      const map = {
        0: "Clear sky",
        1: "Mainly clear",
        2: "Partly cloudy",
        3: "Overcast",
        45: "Fog",
        48: "Depositing rime fog",
        51: "Light drizzle",
        53: "Moderate drizzle",
        55: "Dense drizzle",
        56: "Freezing drizzle",
        57: "Freezing drizzle (dense)",
        61: "Slight rain",
        63: "Moderate rain",
        65: "Heavy rain",
        66: "Freezing rain light",
        67: "Freezing rain heavy",
        71: "Slight snow",
        73: "Moderate snow",
        75: "Heavy snow",
        77: "Snow grains",
        80: "Rain showers light",
        81: "Rain showers moderate",
        82: "Rain showers violent",
        85: "Snow showers slight",
        86: "Snow showers heavy",
        95: "Thunderstorm",
        96: "Thunderstorm with hail (light)",
        99: "Thunderstorm with hail (heavy)"
      };
      return map[code] || "Unknown";
    }

    function renderForecast(cityName, forecastData) {
      document.getElementById("city-name").textContent = cityName;
      const container = document.getElementById("forecast-days");
      container.innerHTML = ""; // পুরনো মুছে দাও

      const daily = forecastData.daily;
      const times = daily.time;  // array of dates (e.g. ["2025-09-27", "2025-09-28", "2025-09-29"])
      const tmax = daily.temperature_2m_max;
      const tmin = daily.temperature_2m_min;
      const wc = daily.weathercode;

      for (let i = 0; i < times.length; i++) {
        const dayDiv = document.createElement("div");
        dayDiv.className = "day-forecast";

        const h2 = document.createElement("h2");
        h2.textContent = times[i];
        dayDiv.appendChild(h2);

        const tempsDiv = document.createElement("div");
        tempsDiv.className = "temps";
        tempsDiv.innerHTML = `<div>Min: ${tmin[i]}°C</div><div>Max: ${tmax[i]}°C</div>`;
        dayDiv.appendChild(tempsDiv);

        const skyDiv = document.createElement("div");
        skyDiv.className = "sky";
        skyDiv.textContent = `Sky: ${weatherCodeToDescription(wc[i])}`;
        dayDiv.appendChild(skyDiv);

        container.appendChild(dayDiv);
      }
    }

    // সার্চ বাটন ও পেজ লোডের ইভেন্ট
    document.getElementById("search-btn").addEventListener("click", async () => {
      const city = document.getElementById("city-input").value.trim();
      if (!city) return;
      try {
        const geo = await geocodeCity(city);
        const forecastData = await fetchForecast(geo.latitude, geo.longitude);
        renderForecast(geo.name, forecastData);
      } catch (err) {
        alert("Error: " + err.message);
      }
    });

    window.addEventListener("load", async () => {
      try {
        const geo = await geocodeCity("Dhaka");
        const forecastData = await fetchForecast(geo.latitude, geo.longitude);
        renderForecast(geo.name, forecastData);
      } catch (err) {
        console.error(err);
      }
    });
  </script>

