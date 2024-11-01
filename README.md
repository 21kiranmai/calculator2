# calculator2

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Enhanced SIP Calculator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.7.0/chart.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
        }

        body {
            padding: 20px;
            background-color: #f5f5f5;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
            text-align: center;
        }

        .input-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 20px;
        }

        .input-group {
            display: flex;
            flex-direction: column;
        }

        label {
            margin-bottom: 5px;
            color: #666;
        }

        input {
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 14px;
        }

        button {
            background-color: #2196F3;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            margin: 20px 0;
        }

        button:hover {
            background-color: #1976D2;
        }

        .chart-container {
            height: 400px;
            margin-bottom: 20px;
            position: relative;
        }

        .wealth-cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .wealth-card {
            background-color: #e3f2fd;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
        }

        .wealth-card.topup {
            background-color: #e8f5e9;
        }

        .wealth-card h3 {
            color: #1976D2;
            font-size: 16px;
            margin-bottom: 10px;
        }

        .wealth-card p {
            color: #333;
            font-size: 20px;
            font-weight: bold;
        }

        .wealth-details {
            margin-top: 10px;
            font-size: 14px;
            color: #666;
        }

        .breakdown-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        .breakdown-table th,
        .breakdown-table td {
            padding: 10px;
            border: 1px solid #ddd;
            text-align: right;
        }

        .breakdown-table th {
            background-color: #f5f5f5;
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Enhanced SIP Calculator</h1>
        
        <div class="input-grid">
            <div class="input-group">
                <label>Monthly SIP Amount (₹):</label>
                <input type="number" id="monthlyAmount" value="10000">
            </div>
            <div class="input-group">
                <label>Expected Annual Return (%):</label>
                <input type="number" id="expectedReturn" value="12">
            </div>
            <div class="input-group">
                <label>Investment Period (Years):</label>
                <input type="number" id="investmentPeriod" value="10">
            </div>
            <div class="input-group">
                <label>Annual Top-up Rate (%):</label>
                <input type="number" id="topupRate" value="10">
            </div>
        </div>

        <button onclick="calculateSIP()">Calculate Investment</button>

        <div class="wealth-cards">
            <div class="wealth-card">
                <h3>Regular SIP Wealth Generation</h3>
                <p id="regularTotalValue">₹0</p>
                <div class="wealth-details">
                    <div>Total Investment: <span id="regularTotalInvestment">₹0</span></div>
                    <div>Additional Wealth: <span id="regularAdditionalWealth">₹0</span></div>
                </div>
            </div>
            <div class="wealth-card topup">
                <h3>Top-up SIP Wealth Generation</h3>
                <p id="topupTotalValue">₹0</p>
                <div class="wealth-details">
                    <div>Total Investment: <span id="topupTotalInvestment">₹0</span></div>
                    <div>Additional Wealth: <span id="topupAdditionalWealth">₹0</span></div>
                </div>
            </div>
        </div>

        <div class="results">
            <div class="chart-container">
                <canvas id="sipChart"></canvas>
            </div>
            <div id="breakdownTable"></div>
        </div>
    </div>

    <script>
        let sipChart = null;

        function formatCurrency(value) {
            return '₹' + value.toLocaleString('en-IN', { maximumFractionDigits: 0 });
        }

        function calculateSIP() {
            const monthlyAmount = parseFloat(document.getElementById('monthlyAmount').value);
            const expectedReturn = parseFloat(document.getElementById('expectedReturn').value);
            const investmentPeriod = parseInt(document.getElementById('investmentPeriod').value);
            const topupRate = parseFloat(document.getElementById('topupRate').value);

            const monthlyRate = expectedReturn / 12 / 100;
            const months = investmentPeriod * 12;

            let regularSIPTotal = 0;
            let topupSIPTotal = 0;
            let regularTotalInvestment = 0;
            let topupTotalInvestment = 0;
            let currentMonthlyAmount = monthlyAmount;
            
            const yearlyData = Array(investmentPeriod).fill().map(() => ({
                regularSIP: { investment: 0, value: 0 },
                topupSIP: { investment: 0, value: 0 }
            }));

            // Calculate monthly values
            for (let month = 1; month <= months; month++) {
                // Regular SIP
                regularTotalInvestment += monthlyAmount;
                regularSIPTotal = (regularSIPTotal + monthlyAmount) * (1 + monthlyRate);

                // Top-up SIP
                topupTotalInvestment += currentMonthlyAmount;
                topupSIPTotal = (topupSIPTotal + currentMonthlyAmount) * (1 + monthlyRate);

                // Update yearly data
                const yearIndex = Math.floor((month - 1) / 12);
                yearlyData[yearIndex].regularSIP.investment += monthlyAmount;
                yearlyData[yearIndex].regularSIP.value = regularSIPTotal;
                yearlyData[yearIndex].topupSIP.investment += currentMonthlyAmount;
                yearlyData[yearIndex].topupSIP.value = topupSIPTotal;

                // Apply annual top-up at the end of each year
                if (month % 12 === 0 && month < months) {
                    currentMonthlyAmount *= (1 + topupRate / 100);
                }
            }

            // Update summary cards
            document.getElementById('regularTotalValue').textContent = formatCurrency(regularSIPTotal);
            document.getElementById('regularTotalInvestment').textContent = formatCurrency(regularTotalInvestment);
            document.getElementById('regularAdditionalWealth').textContent = 
                formatCurrency(regularSIPTotal - regularTotalInvestment);

            document.getElementById('topupTotalValue').textContent = formatCurrency(topupSIPTotal);
            document.getElementById('topupTotalInvestment').textContent = formatCurrency(topupTotalInvestment);
            document.getElementById('topupAdditionalWealth').textContent = 
                formatCurrency(topupSIPTotal - topupTotalInvestment);

            // Update chart
            updateChart(yearlyData);
            // Update table
            updateTable(yearlyData);
        }

        function updateChart(yearlyData) {
            const ctx = document.getElementById('sipChart').getContext('2d');
            
            if (sipChart) {
                sipChart.destroy();
            }

            const labels = yearlyData.map((_, index) => `Year ${index + 1}`);
            
            sipChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                        {
                            label: 'Regular SIP',
                            data: yearlyData.map(data => data.regularSIP.value),
                            backgroundColor: '#1976D2',
                            borderColor: '#1976D2',
                            borderWidth: 1
                        },
                        {
                            label: 'Top-up SIP',
                            data: yearlyData.map(data => data.topupSIP.value),
                            backgroundColor: '#2E7D32',
                            borderColor: '#2E7D32',
                            borderWidth: 1
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                callback: value => formatCurrency(value)
                            }
                        }
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return `${context.dataset.label}: ${formatCurrency(context.raw)}`;
                                }
                            }
                        }
                    }
                }
            });
        }

        function updateTable(yearlyData) {
            let tableHTML = `
                <table class="breakdown-table">
                    <thead>
                        <tr>
                            <th>Year</th>
                            <th>Regular SIP Investment</th>
                            <th>Regular SIP Value</th>
                            <th>Regular Additional Wealth</th>
                            <th>Top-up SIP Investment</th>
                            <th>Top-up SIP Value</th>
                            <th>Top-up Additional Wealth</th>
                        </tr>
                    </thead>
                    <tbody>
            `;

            yearlyData.forEach((data, index) => {
                const regularAdditionalWealth = data.regularSIP.value - data.regularSIP.investment;
                const topupAdditionalWealth = data.topupSIP.value - data.topupSIP.investment;

                tableHTML += `
                    <tr>
                        <td style="text-align: center">Year ${index + 1}</td>
                        <td>${formatCurrency(data.regularSIP.investment)}</td>
                        <td>${formatCurrency(data.regularSIP.value)}</td>
                        <td>${formatCurrency(regularAdditionalWealth)}</td>
                        <td>${formatCurrency(data.topupSIP.investment)}</td>
                        <td>${formatCurrency(data.topupSIP.value)}</td>
                        <td>${formatCurrency(topupAdditionalWealth)}</td>
                    </tr>
                `;
            });

            tableHTML += `
                    </tbody>
                </table>
            `;

            document.getElementById('breakdownTable').innerHTML = tableHTML;
        }

        // Calculate initial values when page loads
        document.addEventListener('DOMContentLoaded', calculateSIP);
    </script>
</body>
</html>
