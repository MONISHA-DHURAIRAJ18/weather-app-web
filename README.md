# weather-app-web
A simple weather app using OpenWeather API.


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Weather</title>
  
  <link rel="icon" href="weather.webp">
  <style>
    body {
      font-family: Arial, sans-serif;
      background: lightblue;
      text-align: center;
      padding: 50px;
      transition: background 0.5s ease;
    }
    .container {
      background: white;
      padding: 30px;
      border-radius: 10px;
      display: inline-block;
      width: 300px;
    }
    input, button {
      padding: 10px;
      margin-bottom: 10px;
      width: 80%;
    }
    .toggle {
      margin-top: 10px;
    }
    #weatherResult, #forecastResult {
      margin-top: 20px;
    }
    .forecast-day {
      margin: 10px 0;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>⛈️Weather</h1>
    <input type="text" id="cityInput" placeholder="Enter city name" />
    <br />
    <button onclick="getWeather()">☂ Get Weather</button>
    <br />
    <button onclick="getWeatherByLocation()">📍 Use My Location</button>
    <br />
    <label class="toggle">
      <input type="checkbox" id="unitToggle" onchange="toggleUnits()" />
      <span>°C / °F</span>
    </label>

    <div id="weatherResult"></div>
    <div id="forecastResult"></div>
  </div>

  <script>
    const apiKey = "237165d3c3444c50568b33f8192d07dc";
    let currentUnit = "metric";

    function getWeather() {
      const city = document.getElementById("cityInput").value.trim();
      if (!city) {
        alert("Please enter a city name.");
        return;
      }

      const weatherURL = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=${currentUnit}`;
      const forecastURL = `https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=${currentUnit}`;
      fetchWeatherData(weatherURL, forecastURL);
    }

    function getWeatherByLocation() {
      if (!navigator.geolocation) {
        alert("Geolocation is not supported by your browser.");
        return;
      }

      navigator.geolocation.getCurrentPosition(
        (position) => {
          const lat = position.coords.latitude;
          const lon = position.coords.longitude;
          const weatherURL = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=${currentUnit}`;
          const forecastURL = `https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&appid=${apiKey}&units=${currentUnit}`;
          fetchWeatherData(weatherURL, forecastURL);
        },
        () => alert("Unable to retrieve your location.")
      );
    }

    function fetchWeatherData(weatherURL, forecastURL) {
      fetch(weatherURL)
        .then((res) => res.json())
        .then((data) => {
          if (data.cod !== 200) throw new Error(data.message);
          
          const temp = data.main.temp;
          const feels_like = data.main.feels_like;
          const description = data.weather[0].description;
          const icon = data.weather[0].icon;
          const humidity = data.main.humidity;
          const wind = data.wind.speed;
          const cityName = data.name;
          const localTime = new Date((data.dt + data.timezone) * 1000).toUTCString();
          const unitSymbol = currentUnit === "metric" ? "°C" : "°F";
          const speedUnit = currentUnit === "metric" ? "m/s" : "mph";

          // Change background based on weather
          changeBackground(description);

          document.getElementById("weatherResult").innerHTML = `
            <h2>${cityName}</h2>
            <img src="http://openweathermap.org/img/wn/${icon}@2x.png" alt="${description}">
            <p><strong>Temperature:</strong> ${temp} ${unitSymbol}</p>
            <p><strong>Feels Like:</strong> ${feels_like} ${unitSymbol}</p>
            <p><strong>Condition:</strong> ${description}</p>
            <p><strong>Humidity:</strong> ${humidity}%</p>
            <p><strong>Wind:</strong> ${wind} ${speedUnit}</p>
            <p><strong>Local Time:</strong> ${localTime}</p>
          `;
        })
        .catch((error) => {
          document.getElementById("weatherResult").innerHTML = `<p style="color:red;">${error.message}</p>`;
        });

      fetch(forecastURL)
        .then((res) => res.json())
        .then((data) => {
          if (data.cod !== "200") throw new Error("Forecast data not found.");
          const forecastBox = document.getElementById("forecastResult");
          const forecasts = data.list.filter(item => item.dt_txt.includes("12:00:00")).slice(0, 3);
          const unitSymbol = currentUnit === "metric" ? "°C" : "°F";

          let forecastHTML = `<h3>3-Day Forecast</h3>`;
          forecasts.forEach(item => {
            const date = new Date(item.dt_txt).toDateString();
            const temp = item.main.temp;
            const icon = item.weather[0].icon;
            const description = item.weather[0].description;

            forecastHTML += `
              <div class="forecast-day">
                <strong>${date}</strong><br>
                <img src="http://openweathermap.org/img/wn/${icon}@2x.png" alt="${description}">
                <p>${temp} ${unitSymbol} - ${description}</p>
              </div>
            `;
          });

          forecastBox.innerHTML = forecastHTML;
        })
        .catch((error) => {
          document.getElementById("forecastResult").innerHTML = `<p style="color:red;">${error.message}</p>`;
        });
    }

    function toggleUnits() {
      currentUnit = document.getElementById("unitToggle").checked ? "imperial" : "metric";
      getWeather(); // Refresh data on toggle
    }

    function changeBackground(condition) {
      condition = condition.toLowerCase();
      if (condition.includes("clear")) {
        document.body.style.background = "#fceabb";
      } else if (condition.includes("cloud")) {
        document.body.style.background = "#dfe6e9";
      } else if (condition.includes("rain") || condition.includes("drizzle")) {
        document.body.style.background = "#a0c4ff";
      } else if (condition.includes("thunderstorm")) {
        document.body.style.background = "#b388ff";
      } else if (condition.includes("snow")) {
        document.body.style.background = "#ffffff";
      } else {
        document.body.style.background = "#e0f7fa";
      }
    }

    // ⏱ Auto-refresh every 15 minutes
    setInterval(() => {
      const city = document.getElementById("cityInput").value.trim();
      if (city) {
        getWeather();
      }
    }, 15 * 60 * 1000); // 15 mins
  </script>
</body>
</html>
