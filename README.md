<!DOCTYPE html>
<html>
<head>
  <title>Weather Knows the 3 C's</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(to top, lightblue, white);
      text-align: center;
      padding: 30px;
    }
    h1 {
      color: steelblue;
    }
    .section {
      margin: 20px 0;
      padding: 15px;
      border: 1px solid lightgray;
      border-radius: 10px;
      background-color: whitesmoke;
      box-shadow: 0 2px 5px rgba(0,0,0,0.05);
    }
    canvas {
      max-width: 600px;
      margin: 0 auto;
    }
  </style>
</head>
<body>

  <h1>🌤️ Weather Knows the 3 C's</h1>
  <center><h2>Visualizing Weather Through Information Design</h2></center>
  <p>A student-focused weather site containing live forecast data so you can plan safer, smarter, and greener.</p>
   
  <div class="section" id="cloud-section">
    <h2>☁️ Cloud Status</h2>
    <p id="cloud-status">Loading...</p>
  </div>

  <div class="section" id="chart-section">
    <h2>📊 Weekly Temperature Chart</h2>
    <canvas id="tempChart" width="400" height="200"></canvas>
  </div>
  
  <div class="section" id="clarity-section">
    <h2>🔍 Clarity Report</h2>
    <p id="clarity-report">Loading...</p>
  </div>

  <script>
const apiKey = "bad60619ca006eb7157e08b43a400527";
const lat = 9.7401;  // Puerto Princesa latitude
const lon = 118.7350; // Puerto Princesa longitude

const clarityMessages = {
  "Clear": "The sky is clear and bright. Perfect for outdoor activities!",
  "Clouds": "Some clouds in the sky, but it’s still a nice day.",
  "Rain": "Rain is expected. Best to stay indoors and safe.",
  "Storm": "Storms expected. Stay safe indoors."
};

// Weekday labels
const labels = [];
const today = new Date();
for(let i = 0; i < 7; i++){
  const day = new Date();
  day.setDate(today.getDate() + i);
  labels.push(day.toLocaleDateString('en-US', { weekday: 'short' }));
}

const ctx = document.getElementById('tempChart').getContext('2d');
const tempChart = new Chart(ctx, {
  type: 'line',
  data: {
    labels: labels,
    datasets: [
      {
        label: "Max Temp (°C)",
        data: [],
        backgroundColor: "red",
        borderColor: "red",
        borderWidth: 2,
        fill: true,
        tension: 0.3,
        pointRadius: 5
      },
      {
        label: "Min Temp (°C)",
        data: [],
        backgroundColor: "blue",
        borderColor: "blue",
        borderWidth: 2,
        fill: true,
        tension: 0.3,
        pointRadius: 5
      }
    ]
  },
  options: {
    responsive: true,
    scales: { y: { beginAtZero: false } }
  }
});

async function fetchWeather() {
  try {
    const url = `https://api.openweathermap.org/data/3.0/onecall?lat=${lat}&lon=${lon}&exclude=minutely,hourly,alerts&units=metric&appid=${apiKey}`;
    const response = await fetch(url);
    if (!response.ok) throw new Error("Weather data unavailable");
    const data = await response.json();

    // Current weather
    const weatherMain = data.daily[0].weather[0].main;
    document.getElementById("cloud-status").textContent = weatherMain;

    let message = clarityMessages["Clouds"];
    if (weatherMain.includes("Clear")) message = clarityMessages["Clear"];
    else if (weatherMain.includes("Rain")) message = clarityMessages["Rain"];
    else if (weatherMain.includes("Storm")) message = clarityMessages["Storm"];
    document.getElementById("clarity-report").textContent = message;

    // 7-day max and min temperatures
    const maxTemps = data.daily.slice(0,7).map(day => day.temp.max);
    const minTemps = data.daily.slice(0,7).map(day => day.temp.min);

    tempChart.data.datasets[0].data = maxTemps;
    tempChart.data.datasets[1].data = minTemps;
    tempChart.update();

  } catch (error) {
    document.getElementById("cloud-status").textContent = "Unable to fetch weather data.";
    document.getElementById("clarity-report").textContent = "Please check your API key or internet connection.";
    console.error("Weather fetch error:", error);
  }
}

fetchWeather();
</script>
