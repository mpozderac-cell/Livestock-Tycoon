<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Livestock Tycoon</title>
  <style>
    body {
      font-family: sans-serif;
      padding: 20px;
      background-color: #f4f4f4;
    }
    h1 {
      color: #2c3e50;
    }
    .section {
      background: white;
      padding: 15px;
      margin-bottom: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    button {
      margin-top: 10px;
      padding: 10px;
      border: none;
      border-radius: 6px;
      background: #3498db;
      color: white;
      cursor: pointer;
    }
    button:hover {
      background: #2980b9;
    }
    select, input[type=text], input[type=range], input[type=number] {
      padding: 8px;
      margin-top: 5px;
      width: 100%;
      box-sizing: border-box;
    }
    label span {
      font-weight: bold;
      display: inline-block;
      margin-top: 10px;
    }
    img {
      max-width: 100px;
      display: block;
      margin-top: 10px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }
    th {
      background-color: #eaeaea;
    }
    #profitChart {
      max-width: 100%;
      height: 250px;
      margin-top: 20px;
    }
  </style>
</head>
<body>
  <h1>Livestock Tycoon</h1>

  <div class="section">
    <label>Farm Name:</label>
    <input type="text" id="farmName" />
    <label>Student Name:</label>
    <input type="text" id="studentName" />
  </div>

  <div class="section">
    <label><span>Choose an Animal:</span></label>
    <select id="animal" onchange="updateAnimalImage()">
      <option value="pig">Pig</option>
      <option value="steer">Steer</option>
      <option value="lamb">Lamb</option>
    </select>
    <img
      id="animalImage"
      src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Young_pig.jpg/320px-Young_pig.jpg"
      alt="Animal"
    />

    <label><span>Quantity:</span></label>
    <input type="number" id="animalQty" min="1" value="1" />

    <label><span>Set Animal Purchase Price ($100–$400):</span></label>
    <input
      type="range"
      id="animalPrice"
      min="100"
      max="400"
      value="150"
      oninput="document.getElementById('animalPriceVal').innerText = this.value"
    />
    <span>Price: $<span id="animalPriceVal">150</span></span>

    <label><span>Set Feed Cost ($10–100):</span></label>
    <input
      type="range"
      id="feedCost"
      min="10"
      max="100"
      value="20"
      oninput="document.getElementById('feedCostVal').innerText = this.value"
    />
    <span>Feed Cost: $<span id="feedCostVal">20</span></span>

    <label><span>Choose Feed Type:</span></label>
    <select id="feedType">
      <option value="corn">Corn</option>
      <option value="alfalfa">Alfalfa</option>
      <option value="soy">Soy Meal</option>
    </select>

    <label><span>Choose a Buyer:</span></label>
    <select id="buyer">
      <option value="auction">Auction House (variable)</option>
      <option value="packer">Packer (stable)</option>
      <option value="direct">Direct-to-Consumer (high for quality)</option>
    </select>

    <button onclick="finishYear()">Finish Year</button>
    <audio id="soundEffect" src="https://www.soundjay.com/button/beep-07.wav" preload="auto"></audio>
  </div>

  <div class="section">
    <h2>Results:</h2>
    <p id="result"></p>
  </div>

  <div class="section">
    <h2>Leaderboard</h2>
    <table id="leaderboard">
      <thead>
        <tr>
          <th>Name</th>
          <th>Farm</th>
          <th>Animal</th>
          <th>Quantity</th>
          <th>Feed</th>
          <th>Buyer</th>
          <th>Year</th>
          <th>Profit</th>
          <th>Total Profit</th>
        </tr>
      </thead>
      <tbody id="leaderboardBody"></tbody>
    </table>
  </div>

  <div class="section">
    <h2>Profit Over 5 Years</h2>
    <canvas id="profitChart"></canvas>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <script>
    const animalImages = {
      pig: "https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Young_pig.jpg/320px-Young_pig.jpg",
      steer:
        "https://upload.wikimedia.org/wikipedia/commons/thumb/e/e1/Cow_female_black_white.jpg/320px-Cow_female_black_white.jpg",
      lamb:
        "https://upload.wikimedia.org/wikipedia/commons/thumb/4/4f/Sheep_and_lamb.jpg/320px-Sheep_and_lamb.jpg",
    };

    const basePrice = { pig: 300, steer: 600, lamb: 200 };
    const feedTypeEffect = { corn: 1.0, alfalfa: 1.1, soy: 1.2 };
    const buyerEffect = {
      auction: () => 0.8 + Math.random() * 0.6,
      packer: () => 1.0,
      direct: () => 1.5,
    };

    let year = 1;
    const maxYears = 5;

    // We'll track profit by student+farm
    // Stored as { year: profit }
    let sessionData = {
      name: "",
      farm: "",
      year: 1,
      profits: [], // index 0 = year 1 profit, etc
      totalProfit: 0,
    };

    // Chart.js setup
    let chart = null;
    function createChart() {
      const ctx = document.getElementById("profitChart").getContext("2d");
      if (chart) {
        chart.destroy();
      }
      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: [...Array(maxYears).keys()].map((i) => `Year ${i + 1}`),
          datasets: [
            {
              label: `Profit for ${sessionData.farm || "Farm"}`,
              data: sessionData.profits,
              borderColor: "rgba(52, 152, 219, 1)",
              backgroundColor: "rgba(52, 152, 219, 0.2)",
              fill: true,
              tension: 0.3,
              pointRadius: 5,
              pointHoverRadius: 7,
            },
          ],
        },
        options: {
          scales: {
            y: {
              beginAtZero: true,
              ticks: {
                stepSize: 100,
              },
            },
          },
          responsive: true,
          plugins: {
            legend: {
              display: true,
            },
          },
        },
      });
    }

    function updateAnimalImage() {
      const animal = document.getElementById("animal").value;
      document.getElementById("animalImage").src = animalImages[animal];
    }

    function getWeatherEffect() {
      const weather = ["sunny", "rainy", "drought", "storm"];
      const selected = weather[Math.floor(Math.random() * weather.length)];
      if (selected === "drought") return 0.8;
      if (selected === "storm") return 0.9;
      return 1.0;
    }

    function getHealthEffect() {
      return 0.8 + Math.random() * 0.4; // 0.8–1.2
    }

    function finishYear() {
      const name = document.getElementById("studentName").value.trim();
      const farm = document.getElementById("farmName").value.trim();
      if (!name || !farm) {
        alert("Please enter your name and farm name.");
        return;
      }

      // On first run or if student/farm changed, reset session data
      if (sessionData.name !== name || sessionData.farm !== farm) {
        sessionData = {
          name,
          farm,
          year: 1,
          profits: [],
          totalProfit: 0,
        };
        // Clear leaderboard table rows
        document.getElementById("leaderboardBody").innerHTML = "";
      }

      const animal = document.getElementById("animal").value;
      const qty = parseInt(document.getElementById("animalQty").value);
      if (qty < 1 || isNaN(qty)) {
        alert("Quantity must be at least 1.");
        return;
      }

      const price = parseInt(document.getElementById("animalPrice").value);
      const feedCost = parseInt(document.getElementById("feedCost").value);
      const buyer = document.getElementById("buyer").value;
      const feedType = document.getElementById("feedType").value;

      document.getElementById("soundEffect").play();

      let feedQuality = 0.5 + feedCost / 100;
      feedQuality *= feedTypeEffect[feedType];

      let quality = basePrice[animal] * feedQuality;
      quality *= getWeatherEffect();
      quality *= getHealthEffect();

      if (buyer === "direct" && feedQuality < 1) quality *= 0.7;

      // Revenue and costs scaled by quantity
      const revenue = quality * buyerEffect[buyer]() * qty;
      const cost = (price + feedCost) * qty;
      const profit = Math.round(revenue - cost);

      sessionData.totalProfit += profit;
      sessionData.profits.push(profit);
      sessionData.year++;

      document.getElementById(
        "result"
      ).innerText = `Year ${sessionData.year - 1}: You made $${Math.round(
        revenue
      )} in sales, spent $${cost}, and profit is $${profit}.\nTotal after ${
        sessionData.year - 1
      } year(s): $${sessionData.totalProfit}`;

      // Add row to leaderboard
      const row = document.createElement("tr");
      row.innerHTML = `
        <td>${name}</td>
        <td>${farm}</td>
        <td>${animal}</td>
        <td>${qty}</td>
        <td>${feedType}</td>
        <td>${buyer}</td>
        <td>${sessionData.year - 1}</td>
        <td>$${profit}</td>
        <td>$${sessionData.totalProfit}</td>
      `;
      document.getElementById("leaderboardBody").appendChild(row);

      // Save data in localStorage per student+farm
      const key = `livestockData_${name}_${farm}`;
      localStorage.setItem(key, JSON.stringify(sessionData));

      createChart();

      if (sessionData.year > maxYears) {
        alert(`Game over! Total profit after ${maxYears} years: $${sessionData.totalProfit}`);
        sessionData.year = 1;
        sessionData.profits = [];
        sessionData.totalProfit = 0;
        // Clear leaderboard for new game
        document.getElementById("leaderboardBody").innerHTML = "";
        createChart();
      }
    }

    // On load, try to restore previous session data if possible
    window.onload = function () {
      const name = document.getElementById("studentName").value.trim();
      const farm = document.getElementById("farmName").value.trim();

      // Optional: you can add a small UI to load saved sessions for name/farm combos,
      // but here we just initialize an empty chart
      createChart();
    };
  </script>
</body>
</html>
