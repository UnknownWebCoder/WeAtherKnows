<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Weather Knows the 3 C's</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(to top, #ccefff, #ffffff);
      text-align: center;
      padding: 30px;
      color: #333;
    }
    h1 { color: #2b6cb0; }
    nav a {
      margin: 0 12px;
      text-decoration: none;
      color: #2b6cb0;
      font-weight: bold;
      transition: color 0.3s;
    }
    nav a:hover { color: #1a4f8b; }
    .section {
      margin: 25px auto;
      padding: 15px;
      width: 80%;
      max-width: 700px;
      border: 1px solid #ddd;
      border-radius: 10px;
      background: #f8f9fa;
      box-shadow: 0 2px 6px rgba(0,0,0,0.05);
    }
    canvas { max-width: 600px; margin: 0 auto; }

    // Collapsible About Section
    #about-content {
      max-height: 400px;
      overflow: hidden;
      transition: max-height 0.6s ease, opacity 0.6s ease;
      opacity: 1;
    }
    #about-content.hidden {
      max-height: 0;
      opacity: 0;
    }
    #About h2 {
      cursor: pointer;
      user-select: none;
      color: #2b6cb0;
    }
    #temp-indicator {
      font-size: 1.5em;
      margin-top: 10px;
      font-weight: bold;
      transition: color 0.3s;
    }
  </style>
</head>
<body>

  <h1>🌤️ Weather Knows the 3 C's</h1>
  <h2>Visualizing Weather Through Information Design</h2>
  <p>A student-focused weather site with live forecasts so you can plan safer, smarter, and greener.</p>

  <nav>
    <a href="#About">About</a> |
    <a href="#cloud-section">Clouds</a> |
    <a href="#chart-section">Temperature</a> |
    <a href="#clarity-section">Clarity</a>
  </nav>

  <div class="section" id="About">
    <h2 onclick="toggleAbout()">ℹ️ About ▼</h2>
    <div id="about-content">
      <p>
        "Weather Knows the 3 C’s" is a student-friendly weather site that visualizes
        <strong>Clouds, Climate, and Clarity</strong> to help you make informed, sustainable decisions.
      </p>
    </div>
  </div>

  <div class="section" id="cloud-section">
    <h2>☁️ Cloud Status</h2>
    <p id="cloud-status">Loading...</p>
  </div>

  <div class="section" id="chart-section">
    <h2>📊 Real-Time Temperature Chart</h2>
    <canvas id="tempChart" width="400" height="200"></canvas>
    <div id="temp-indicator">Temperature: -- °C</div>
  </div>

  <div class="section" id="clarity-section">
    <h2>🔍 Clarity Report</h2>
    <p id="clarity-report">Loading...</p>
  </div>

  <script>
    const apiKey = "6214843141da455fa6c235320252610";
    const city = "Puerto Princesa";

    const clarityMsg = {
      Clear: "The sky is clear and bright. Perfect for outdoor activities!",
      "Partly Cloudy": "Some clouds, but still a pleasant day.",
      Cloudy: "The sky is covered — a bit gloomy.",
      Stormy: "Storms expected. Stay safe indoors."
    };

    // Chart setup
    const ctx = document.getElementById("tempChart").getContext("2d");
    const chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: [],
        datasets: [{
          label: "Real-Time Temperature (°C)",
          data: [],
          borderColor: "#1E90FF",
          backgroundColor: "rgba(135,206,250,0.3)",
          fill: true,
          tension: 0.4
        }]
      },
      options: {
        animation: { duration: 500 },
        scales: { y: { beginAtZero: false } }
      }
    });

    // Fetch real-time temperature
    async function fetchRealTimeWeather() {
      try {
        const url = `https://api.worldweatheronline.com/premium/v1/weather.ashx?key=${apiKey}&q=${city}&format=json`;
        const res = await fetch(url);
        if (!res.ok) throw new Error("Weather data unavailable");
        const data = await res.json();

        const current = data.data.current_condition[0];
        const temp = parseFloat(current.temp_C);
        const desc = current.weatherDesc[0].value;

        // Update texts
        document.getElementById("cloud-status").textContent = desc;

        const clarity = desc.includes("Clear") ? clarityMsg.Clear :
                        desc.includes("Partly") ? clarityMsg["Partly Cloudy"] :
                        desc.includes("Rain") || desc.includes("Storm") ? clarityMsg.Stormy :
                        clarityMsg.Cloudy;
        document.getElementById("clarity-report").textContent = clarity;

        // Update chart
        const now = new Date().toLocaleTimeString();
        chart.data.labels.push(now);
        chart.data.datasets[0].data.push(temp);
        if (chart.data.labels.length > 10) {
          chart.data.labels.shift();
          chart.data.datasets[0].data.shift();
        }
        chart.update();

        // Update live temperature indicator
        const tempDisplay = document.getElementById("temp-indicator");
        tempDisplay.textContent = `Temperature: ${temp.toFixed(1)} °C`;
        tempDisplay.style.color = temp > 30 ? "red" : temp < 25 ? "blue" : "green";

      } catch (err) {
        document.getElementById("cloud-status").textContent = "Unable to fetch data.";
        console.error("Fetch error:", err);
      }
    }

    // Fetch every 30 seconds
    fetchRealTimeWeather();
    setInterval(fetchRealTimeWeather, 30000);

    // Collapsible About Section
    function toggleAbout() {
      const about = document.getElementById("about-content");
      const header = document.querySelector("#About h2");
      about.classList.toggle("hidden");
      header.innerHTML = about.classList.contains("hidden") ? "ℹ️ About ▶" : "ℹ️ About ▼";
    }
  </script>
</body>
</html>
