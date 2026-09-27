# Simple-screener
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Screener Saham Browser</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 15px; background: #f4f6f8; margin: 0; }
        h2 { color: #1e293b; text-align: center; }
        .card { background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-bottom: 20px; }
        button { background: #2563eb; color: white; border: none; padding: 12px 15px; border-radius: 5px; width: 100%; font-size: 16px; cursor: pointer; font-weight: bold; }
        button:disabled { background: #94a3b8; }
        table { width: 100%; border-collapse: collapse; margin-top: 15px; font-size: 12px; }
        th, td { border: 1px solid #cbd5e1; padding: 8px; text-align: center; }
        th { background: #1e293b; color: white; }
        .buy { background: #dcfce7; color: #166534; font-weight: bold; }
        .sell { background: #fee2e2; color: #991b1b; font-weight: bold; }
        .hold { background: #fef3c7; color: #92400e; font-weight: bold; }
    </style>
</head>
<body>

    <h2>📊 Stock Screener & Fundamental Analysis</h2>

    <div class="card">
        <button id="btnScan" onclick="jalankanScreener()">🔍 Jalankan Analisis</button>
    </div>

    <div class="card" style="overflow-x: auto;">
        <table id="tabelHasil">
            <thead>
                <tr>
                    <th>Ticker</th>
                    <th>Harga</th>
                    <th>PER</th>
                    <th>PBV</th>
                    <th>Rekomendasi</th>
                </tr>
            </thead>
            <tbody>
                <tr><td colspan="5">Tekan tombol di atas untuk memuat data...</td></tr>
            </tbody>
        </table>
    </div>

    <script>
        // Gunakan API Key gratisan dari site FMP (financialmodelingprep.com)
        // 'demo' hanya berlaku untuk contoh; ganti dengan API Key gratis Anda jika ingin saham Indonesia (.JK)
        const API_KEY = "demo"; 
        const DAFTAR_SAHAM = ["AAPL", "MSFT", "GOOGL"]; // Untuk pengujian stabil pertama kali

        async function ambilDataLengkap(ticker) {
            const urlProfile = `https://financialmodelingprep.com/api/v3/profile/${ticker}?apikey=${API_KEY}`;
            const urlRatios = `https://financialmodelingprep.com/api/v3/ratios-ttm/${ticker}?apikey=${API_KEY}`;

            try {
                const resProfile = await fetch(urlProfile);
                const dataProfile = await resProfile.json();

                const resRatios = await fetch(urlRatios);
                const dataRatios = await resRatios.json();

                if (!dataProfile.length) return null;

                const prof = dataProfile[0];
                const rat = dataRatios[0] || {};

                const harga = prof.price || 0;
                const per = rat.peRatioTTM ? rat.peRatioTTM.toFixed(2) : "N/A";
                const pbv = rat.priceToBookRatioTTM ? rat.priceToBookRatioTTM.toFixed(2) : "N/A";

                // Logika Sederhana
                let rekomendasi = "HOLD";
                let badgeClass = "hold";

                if (per !== "N/A" && parseFloat(per) < 20) {
                    rekomendasi = "BUY";
                    badgeClass = "buy";
                } else if (per !== "N/A" && parseFloat(per) > 40) {
                    rekomendasi = "SELL";
                    badgeClass = "sell";
                }

                return {
                    ticker: ticker,
                    harga: "$" + harga.toFixed(2),
                    per: per,
                    pbv: pbv,
                    rekomendasi: rekomendasi,
                    badgeClass: badgeClass
                };
            } catch (error) {
                console.error("Gagal mengambil data " + ticker, error);
                return null;
            }
        }

        async function jalankanScreener() {
            const btn = document.getElementById("btnScan");
            const tbody = document.querySelector("#tabelHasil tbody");
            
            btn.disabled = true;
            btn.innerText = "Memproses Data Pasar...";
            tbody.innerHTML = "<tr><td colspan='5'>Sedang memuat data dari API...</td></tr>";

            let htmlRows = "";

            for (let ticker of DAFTAR_SAHAM) {
                const data = await ambilDataLengkap(ticker);
                if (data) {
                    htmlRows += `
                        <tr>
                            <td><b>${data.ticker}</b></td>
                            <td>${data.harga}</td>
                            <td>${data.per}</td>
                            <td>${data.pbv}</td>
                            <td class="${data.badgeClass}">${data.rekomendasi}</td>
                        </tr>
                    `;
                }
            }

            tbody.innerHTML = htmlRows || "<tr><td colspan='5'>Gagal mengambil data. Pastikan jaringan terhubung.</td></tr>";
            btn.disabled = false;
            btn.innerText = "🔍 Jalankan Analisis";
        }
    </script>
</body>
</html>
