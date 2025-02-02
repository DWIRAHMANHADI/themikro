## Mikrotik _feat_ Fonnte 🚀
Connect Mikrotik API With ChartJS for status realtime trafic user PPPOE.

🎖️ **Tested on Mikrotik v.7.0**


Download copy and paste routeros_api.class.php on your folder
```
https://github.com/BenMenking/routeros-api/blob/master/routeros_api.class.php
```

Create file php name bandwidth_pppoe.php :heavy_check_mark:
```
<?php
require('routeros_api.class.php');

$API = new RouterosAPI();

$username = isset($_GET['username']) ? $_GET['username'] : ''; // GET username

if ($API->connect('192.168.88.1', 'admin', 'password', port)) { 

    $API->write('/ppp/active/print');
    $users = $API->read(false);
    $ppp_users = $API->parseResponse($users);

    $data = [];

    // search username
    foreach ($ppp_users as $user) {
        if (strpos($user['name'], $username) !== false) {
            $interface = $user['name'];

            // Memantau trafik per user PPPoE
            $API->write('/interface/monitor-traffic', false);
            $API->write("=interface=<pppoe-$interface>", false);
            $API->write("=once=", true);
            $traffic = $API->read(false);
            $traffic_data = $API->parseResponse($traffic);

            if (!empty($traffic_data)) {
                // Konversi dari bps ke Mbps
                $rx_bps = isset($traffic_data[0]['rx-bits-per-second']) ? $traffic_data[0]['rx-bits-per-second'] : 0;
                $tx_bps = isset($traffic_data[0]['tx-bits-per-second']) ? $traffic_data[0]['tx-bits-per-second'] : 0;

                // Convert bps to Mbps
                $rx_mbps = $rx_bps / 1000000; // Rx dalam Mbps
                $tx_mbps = $tx_bps / 1000000; // Tx dalam Mbps

                $data[] = [
                    'username' => $user['name'],
                    'rx' => $rx_mbps,
                    'tx' => $tx_mbps
                ];
            }
        }
    }

    $API->disconnect();

    if (empty($data)) {
        echo json_encode(["error" => "Username not found"]);
    } else {
        header('Content-Type: application/json');
        echo json_encode(['data' => $data]);
    }
} else {
    echo json_encode(["error" => "failed Connect Mikrotik"]);
}
?>
```
Create file index.html on the root folder :x:
```
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grafik Trafik PPPoE per User dengan Pencarian</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            height: 100vh;
            background-color: #f4f4f9;
        }

        h2 {
            margin-top: 20px;
            text-align: center;
        }

        #usernameSearch {
            margin-top: 20px;
            padding: 10px;
            width: 80%;
            max-width: 300px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 4px;
            text-align: center;
        }

        #userChartsContainer {
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            flex-direction: column;
            align-items: center;
            flex-grow: 1;
        }

        canvas {
            width: 100% !important;  
            height: 100vh !important; 
        }

        
        .input-container {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            width: 100%;
        }
    </style>
</head>
<body>

    <!-- Container untuk judul dan input pencarian -->
    <div class="input-container">
        <h2>Grafik Trafik Real-Time PPPoE per User</h2>

        <!-- Input pencarian username -->
        <input type="text" id="usernameSearch" placeholder="Cari Username" oninput="searchUser()" />
    </div>

    <!-- Container untuk grafik user -->
    <div id="userChartsContainer"></div>

    <script>
        let userChart = null;
        let intervalID = null;

       
        async function searchUser() {
            const username = document.getElementById('usernameSearch').value.trim();

            
            if (username) {
                await updateChart(username);
            } else {
                document.getElementById("userChartsContainer").innerHTML = ""; // Kosongkan jika tidak ada input
            }
        }

       
        async function updateChart(username) {
            try {
                const response = await fetch(`bandwidth_pppoe.php?username=${username}`);
                const data = await response.json();

                if (!data || !data.data || data.data.length === 0) {
                    alert("Pengguna tidak ditemukan!");
                    return;
                }

                
                document.getElementById("userChartsContainer").innerHTML = "";

                const user = data.data[0]; // Ambil data pengguna pertama

                const canvasId = `chartUser`;
                const canvas = document.createElement("canvas");
                canvas.id = canvasId;
                document.getElementById("userChartsContainer").appendChild(canvas);

                let ctx = document.getElementById(canvasId).getContext('2d');

                userChart = new Chart(ctx, {
                    type: 'line', // Grafik garis
                    data: {
                        labels: [], // Waktu atau interval
                        datasets: [
                            {
                                label: `${user.username} - Upload (Mbps)`,
                                data: [],
                                borderColor: 'blue',
                                fill: false,
                                tension: 0.4,
                                cubicInterpolationMode: 'monotone'
                            },
                            {
                                label: `${user.username} - Download (Mbps)`,
                                data: [],
                                borderColor: 'red',
                                fill: false,
                                tension: 0.4,
                                cubicInterpolationMode: 'monotone'
                            }
                        ]
                    },
                    options: {
                        responsive: true,
                        animation: {
                            duration: 1000, // Durasi animasi setiap update
                            easing: 'easeInOutQuad' // Easing untuk efek transisi progresif
                        },
                        scales: {
                            y: {
                                beginAtZero: true
                            }
                        },
                        plugins: {
                            tooltip: {
                                callbacks: {
                                    // Menambahkan informasi detail saat hover pada grafik
                                    label: function(tooltipItem) {
                                        let label = tooltipItem.dataset.label || '';
                                        if (label) {
                                            label += ': ';
                                        }
                                        label += tooltipItem.raw.toFixed(2) + ' Mbps';
                                        return label;
                                    }
                                }
                            }
                        },
                        
                        plugins: {
                            zoom: {
                                pan: {
                                    enabled: true,
                                    mode: 'xy'
                                },
                                zoom: {
                                    enabled: true,
                                    mode: 'xy',
                                    speed: 0.1
                                }
                            }
                        }
                    }
                });

                
                updateRealTimeData(user);
                
            } catch (error) {
                console.error("Gagal mengambil data:", error);
            }
        }

        
        function updateRealTimeData(user) {
            if (intervalID) {
                clearInterval(intervalID); // Hentikan interval sebelumnya jika ada
            }

            intervalID = setInterval(async () => {
                try {
                    const response = await fetch(`bandwidth_pppoe.php?username=${user.username}`);
                    const data = await response.json();

                    if (!data || !data.data || data.data.length === 0) {
                        alert("Data trafik tidak ditemukan!");
                        return;
                    }

                    const updatedUser = data.data[0];

                    
                    userChart.data.labels.push(new Date().toLocaleTimeString());
                    userChart.data.datasets[0].data.push(updatedUser.rx); // Download (Mbps)
                    userChart.data.datasets[1].data.push(updatedUser.tx); // Upload (Mbps)

                    
                    if (userChart.data.labels.length > 30) {
                        userChart.data.labels.shift();
                        userChart.data.datasets[0].data.shift();
                        userChart.data.datasets[1].data.shift();
                    }

                    userChart.update();
                } catch (error) {
                    console.error("Gagal mengambil data untuk update:", error);
                }
            }, 1000); 
        }
    </script>
</body>
</html>

```

:pushpin: __Spesial Noted in Script__
| Variable     | Information |
|:---------|:----|
|**connect('192.168.88.1', 'admin', 'password', port))**|change this query

