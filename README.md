[script.js](https://github.com/user-attachments/files/29680076/script.js)
let currentPrices = {
    bitcoin: { usd: 0, eur: 0, uah: 0 },
    ethereum: { usd: 0, eur: 0, uah: 0 },
    binancecoin: { usd: 0, eur: 0, uah: 0 },
    solana: { usd: 0, eur: 0, uah: 0 },
    ripple: { usd: 0, eur: 0, uah: 0 }
};

let priceChart = null;
let currentMarketView = 'table';
let currentCoin = 'bitcoin';
let currentTimeframe = '7';
const targetCoins = ['bitcoin', 'ethereum', 'binancecoin', 'solana', 'ripple'];

const coinDetails = {
    bitcoin: { name: "Bitcoin", sym: "btc", index: 1, cap: "$1,350,230,000" },
    ethereum: { name: "Ethereum", sym: "eth", index: 2, cap: "$412,450,000" },
    binancecoin: { name: "BNB", sym: "bnb", index: 3, cap: "$87,120,000" },
    solana: { name: "Solana", sym: "sol", index: 4, cap: "$64,890,000" },
    ripple: { name: "Ripple", sym: "xrp", index: 5, cap: "$31,400,000" }
};

const defaultSignals = [
    { coin: "BTC", sentiment: "Bullish", emoji: "🐳", text: "Інституційні об'єми утримують рівень підтримки. Очікую вихід вгору.", date: "24 травня 2026" },
    { coin: "ETH", sentiment: "Bullish", emoji: "🦊", text: "Стейкінг росте, пропозиція на біржах падає. Сильний сигнал на купівлю.", date: "24 травня 2026" },
    { coin: "SOL", sentiment: "Bearish", emoji: "🐻", text: "Локальний індикатор RSI перегрітий на 4-годинному таймфреймі. Можлива корекція.", date: "23 травня 2026" }
];

async function fetchCryptoData() {
    try {
        const response = await fetch(`https://api.coingecko.com/api/v3/simple/price?ids=${targetCoins.join(',')}&vs_currencies=usd,eur,uah&include_24hr_change=true`);
        const data = await response.json();
        
        targetCoins.forEach(coin => {
            if (data[coin]) {
                currentPrices[coin] = data[coin];
            }
        });

        updateGlobalStats();
        renderMarketContent();
        updateHotCoins();
        calculateConversion();
    } catch (error) {
        console.error(error);
    }
}

function updateGlobalStats() {
    document.getElementById("global-market-cap").textContent = "$2.48T";
}

function updateHotCoins() {
    const container = document.getElementById("hot-coins-container");
    if (!container) return;
    container.innerHTML = "";

    const hotCoins = ['bitcoin', 'ethereum', 'solana'];
    hotCoins.forEach(coin => {
        const info = coinDetails[coin];
        const rates = currentPrices[coin];
        if (!rates) return;
        
        const price = rates.usd;
        const change = rates.usd_24h_change || 0;
        const changeClass = change >= 0 ? "rate-up" : "rate-down";
        const changeSign = change >= 0 ? "+" : "";

        container.innerHTML += `
            <div class="hot-coin-row" onclick="selectCoinAndScroll('${coin}')">
                <div class="hot-coin-info">
                    <strong>${info.name}</strong>
                    <span class="hot-coin-symbol">${info.sym}</span>
                </div>
                <div>
                    <div class="hot-coin-price">$${price.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</div>
                    <div class="${changeClass}" style="font-size: 12px; text-align: right;">${changeSign}${change.toFixed(2)}%</div>
                </div>
            </div>
        `;
    });
}

function renderMarketContent() {
    renderTableView();
    renderGridView();
}

function renderTableView() {
    const tbody = document.getElementById("crypto-table-body");
    if (!tbody) return;
    tbody.innerHTML = "";

    targetCoins.forEach(coin => {
        const rates = currentPrices[coin];
        if (!rates) return;
        
        const info = coinDetails[coin];
        const price = rates.usd;
        const change = rates.usd_24h_change || 0;
        const changeClass = change >= 0 ? "rate-up" : "rate-down";
        const changeSign = change >= 0 ? "+" : "";

        tbody.innerHTML += `
            <tr onclick="updateChartData('${coin}', currentTimeframe)">
                <td>${info.index}</td>
                <td class="coin-cell-name">${info.name} <span class="coin-cell-symbol">${info.sym}</span></td>
                <td class="td-mono">$${price.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</td>
                <td class="${changeClass}">${changeSign}${change.toFixed(2)}%</td>
                <td class="td-mono" style="color: var(--text-muted); font-size: 14px;">${info.cap}</td>
            </tr>
        `;
    });
}

function renderGridView() {
    const grid = document.getElementById("market-grid-view");
    if (!grid) return;
    grid.innerHTML = "";

    targetCoins.forEach((coin, i) => {
        const rates = currentPrices[coin];
        if (!rates) return;

        const info = coinDetails[coin];
        const price = rates.usd;
        const change = rates.usd_24h_change || 0;
        const changeClass = change >= 0 ? "rate-up" : "rate-down";
        const changeSign = change >= 0 ? "+" : "";

        grid.innerHTML += `
            <div class="market-coin-card" style="animation-delay:${i * 0.05}s" onclick="updateChartData('${coin}', currentTimeframe)">
                <div class="card-coin-head">
                    <span class="card-coin-title">${info.name}</span>
                    <span class="card-coin-symbol">${info.sym}</span>
                </div>
                <div class="card-coin-price">$${price.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</div>
                <div class="card-coin-footer">
                    <span style="color: var(--text-muted)">Змена 24г:</span>
                    <span class="${changeClass}">${changeSign}${change.toFixed(2)}%</span>
                </div>
            </div>
        `;
    });
}

function startLiveMarketTicker() {
    setInterval(() => {
        targetCoins.forEach(coin => {
            if (currentPrices[coin] && currentPrices[coin].usd > 0) {
                const randomDeviation = 1 + (Math.random() * 0.0006 - 0.0003);
                
                currentPrices[coin].usd *= randomDeviation;
                currentPrices[coin].eur *= randomDeviation;
                currentPrices[coin].uah *= randomDeviation;
                
                currentPrices[coin].usd_24h_change += (Math.random() * 0.02 - 0.01);
            }
        });
        
        renderMarketContent();
        updateHotCoins();
        calculateConversion();
    }, 3000);
}

async function updateChartData(coinId, timeframe) {
    currentCoin = coinId;
    currentTimeframe = timeframe.toString();
    
    const timeframeTexts = { '1': 'за останні 24 години (погодинно)', '7': 'за останні 7 днів', '30': 'за останні 30 днів', '365': 'за останній рік' };
    const subtitleElement = document.querySelector(".chart-subtitle");
    if (subtitleElement) {
        subtitleElement.textContent = `Динаміка ринкової вартості ${timeframeTexts[currentTimeframe]} за даними API`;
    }

    try {
        let url = `https://api.coingecko.com/api/v3/coins/${coinId}/market_chart?vs_currency=usd&days=${currentTimeframe}`;
        if (currentTimeframe === '1') {
            url += '&interval=hourly';
        } else {
            url += '&interval=daily';
        }

        const response = await fetch(url);
        const data = await response.json();
        
        const prices = data.prices.map(p => p[1]);
        const labels = data.prices.map(p => {
            const date = new Date(p[0]);
            if (currentTimeframe === '1') {
                return date.toLocaleTimeString("uk-UA", { hour: '2-digit', minute: '2-digit' });
            }
            return date.toLocaleDateString("uk-UA", { day: 'numeric', month: 'short' });
        });

        const names = { bitcoin: "Bitcoin (BTC)", ethereum: "Ethereum (ETH)", binancecoin: "BNB", solana: "Solana (SOL)", ripple: "Ripple (XRP)" };
        document.getElementById("chart-title").textContent = `Інтерактивний тренд: ${names[coinId]}`;
        document.getElementById("chart-coin-select").value = coinId;

        renderChart(labels, prices);
    } catch (error) {
        console.error(error);
    }
}

function renderChart(labels, dataPoints) {
    const ctx = document.getElementById('cryptoTrendChart').getContext('2d');
    if (priceChart) {
        priceChart.destroy();
    }

    priceChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                data: dataPoints,
                borderColor: '#3861fb',
                backgroundColor: 'rgba(56, 97, 251, 0.03)',
                borderWidth: 3,
                pointBackgroundColor: '#3861fb',
                pointRadius: labels.length > 50 ? 0 : 3,
                pointHoverRadius: 6,
                fill: true,
                tension: 0.15
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
                x: { grid: { color: '#222f4d' }, ticks: { color: '#8899a6', maxTicksLimit: 12, font: { family: 'DM Sans', size: 11 } } },
                y: { grid: { color: '#222f4d' }, ticks: { color: '#8899a6', font: { family: 'Space Mono', size: 11 } } }
            }
        }
    });
}

function changeChartTimeframe(timeframe) {
    currentTimeframe = timeframe.toString();
    
    document.querySelectorAll(".tf-btn").forEach(btn => btn.classList.remove("active"));
    document.getElementById(`tf-${currentTimeframe}`).classList.add("active");
    
    updateChartData(currentCoin, currentTimeframe);
}

function switchMarketView(viewType) {
    currentMarketView = viewType;
    const tableView = document.getElementById("market-table-view");
    const gridView = document.getElementById("market-grid-view");
    const btnTable = document.getElementById("view-btn-table");
    const btnGrid = document.getElementById("view-btn-grid");

    if (viewType === 'table') {
        tableView.classList.remove("hidden");
        gridView.classList.add("hidden");
        btnTable.classList.add("active");
        btnGrid.classList.remove("active");
    } else {
        tableView.classList.add("hidden");
        gridView.classList.remove("hidden");
        btnTable.classList.remove("remove");
        btnGrid.classList.add("active");
    }
}

function selectCoinAndScroll(coinId) {
    updateChartData(coinId, currentTimeframe);
    document.getElementById("analytics").scrollIntoView({ behavior: 'smooth' });
}

function calculateConversion() {
    const amount = parseFloat(document.getElementById("convert-amount").value) || 0;
    const fromCoin = document.getElementById("convert-from").value;
    const toCurrency = document.getElementById("convert-to").value;

    const coinRates = currentPrices[fromCoin];
    if (!coinRates || !coinRates[toCurrency]) return;

    const rate = coinRates[toCurrency];
    const finalResult = amount * rate;

    const currencySymbols = { usd: "$", eur: "€", uah: "₴" };
    const symbol = currencySymbols[toCurrency];

    document.getElementById("conversion-result").textContent = 
        `${symbol} ${finalResult.toLocaleString("uk-UA", { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
}

function initTimelineInteractivity() {
    const steps = document.querySelectorAll(".timeline-step");
    steps.forEach(step => {
        step.addEventListener("mouseenter", () => step.classList.add("active"));
        step.addEventListener("mouseleave", () => step.classList.remove("active"));
    });
}

function toggleFaq(trigger) {
    const item = trigger.parentElement;
    const content = item.querySelector(".faq-content");
    const isOpen = item.classList.contains("open");

    document.querySelectorAll(".faq-item").forEach(i => {
        i.classList.remove("open");
        i.querySelector(".faq-content").style.maxHeight = "0";
    });

    if (!isOpen) {
        item.classList.add("open");
        content.style.maxHeight = content.scrollHeight + "px";
    }
}

function openSignalModal() { document.getElementById("signal-modal").classList.remove("hidden"); }
function closeSignalModal() { document.getElementById("signal-modal").classList.add("hidden"); }

function loadSignals() {
    let stored = localStorage.getItem("crypto_signals");
    let signals = stored ? JSON.parse(stored) : defaultSignals;
    
    const feed = document.getElementById("signals-feed");
    if (!feed) return;
    feed.innerHTML = "";

    signals.forEach((s, index) => {
        const badgeClass = s.sentiment.toLowerCase() === 'bullish' ? 'bullish' : 'bearish';
        feed.innerHTML += `
            <div class="signal-feed-item" style="position: relative;">
                <button onclick="deleteSignal(${index})" style="position: absolute; top: 12px; right: 12px; background: transparent; border: none; color: var(--text-muted); cursor: pointer; font-size: 12px; transition: color 0.2s;" onmouseenter="this.style.color='var(--crypto-down)'" onmouseleave="this.style.color='var(--text-muted)'" title="Видалити прогноз">✕</button>
                <div class="signal-item-head">
                    <div class="signal-user">
                        <span class="signal-user-avatar">${s.emoji}</span>
                        <span>Аналітик [${s.coin}]</span>
                    </div>
                    <span class="signal-badge ${badgeClass}">${s.sentiment}</span>
                </div>
                <p class="signal-item-text">${s.text}</p>
                <div style="font-size:11px; color:var(--text-muted); text-align:right; margin-top:8px;">${s.date}</div>
            </div>
        `;
    });

    calculateSentiment(signals);
}

function calculateSentiment(signals) {
    const total = signals.length;
    const bullishCount = signals.filter(s => s.sentiment === "Bullish").length;
    const percentage = total > 0 ? Math.round((bullishCount / total) * 100) : 75;

    document.getElementById("sentiment-percentage").textContent = `${percentage}%`;
    document.getElementById("count-bullish").textContent = `${percentage}%`;
    document.getElementById("count-bearish").textContent = `${100 - percentage}%`;
    
    const bars = document.querySelectorAll(".bar-fill");
    if(bars.length >= 2) {
        bars[0].style.width = `${percentage}%`;
        bars[1].style.width = `${100 - percentage}%`;
    }
}

function deleteSignal(index) {
    let stored = localStorage.getItem("crypto_signals");
    let signals = stored ? JSON.parse(stored) : [...defaultSignals];
    signals.splice(index, 1);
    localStorage.setItem("crypto_signals", JSON.stringify(signals));
    loadSignals();
}

function submitNewSignal(event) {
    event.preventDefault();
    const coin = document.getElementById("modal-coin").value;
    const sentiment = document.querySelector('input[name="modal-sentiment"]:checked').value;
    const emoji = document.querySelector('input[name="modal-emoji"]:checked').value;
    const text = document.getElementById("modal-text").value;
    const date = "24 травня 2026";

    let stored = localStorage.getItem("crypto_signals");
    let signals = stored ? JSON.parse(stored) : [...defaultSignals];

    signals.unshift({ coin, sentiment, emoji, text, date });
    localStorage.setItem("crypto_signals", JSON.stringify(signals));

    document.getElementById("modal-signal-form").reset();
    closeSignalModal();
    loadSignals();
    showToast();
}

function showToast() {
    const toast = document.getElementById("toast-notification");
    toast.classList.remove("hidden");
    setTimeout(() => toast.classList.add("hidden"), 3000);
}

function validateEmailField() {
    const input = document.getElementById("user-email");
    const error = document.getElementById("email-error");
    const value = input.value.trim();
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

    if (value === "" || regex.test(value)) {
        input.classList.remove("input-error");
        error.classList.add("hidden");
        return true;
    } else {
        input.classList.add("input-error");
        error.classList.remove("hidden");
        return false;
    }
}

function validatePhoneField() {
    const input = document.getElementById("user-phone");
    const error = document.getElementById("phone-error");
    const value = input.value.trim();
    const regex = /^\+[0-9]{9,14}$/;

    if (value === "" || regex.test(value)) {
        input.classList.remove("input-error");
        error.classList.add("hidden");
        return true;
    } else {
        input.classList.add("input-error");
        error.classList.remove("hidden");
        return false;
    }
}

function handleFormSubmit(event) {
    event.preventDefault();
    if (validateEmailField() && validatePhoneField()) {
        alert("Запит успішно верифіковано та направлено на аналітичний сервер.");
        document.getElementById("analytics-request-form").reset();
    }
}

function toggleTheme() {
    document.documentElement.classList.toggle("light");
    const isLight = document.documentElement.classList.contains("light");
    localStorage.setItem("theme", isLight ? "light" : "dark");
}

function initTheme() {
    const saved = localStorage.getItem("theme");
    if (saved === "light") {
        document.documentElement.classList.add("light");
    } else {
        document.documentElement.classList.remove("light");
    }
}

window.addEventListener("DOMContentLoaded", () => {
    initTheme();
    fetchCryptoData();
    updateChartData('bitcoin', '7');
    loadSignals();
    initTimelineInteractivity();
    startLiveMarketTicker();
    setInterval(fetchCryptoData, 60000);
});
