# Script MQL5 (Otomatis Eksekusi Order)
Berikut adalah script **MQL5** untuk MetaTrader 5 yang secara otomatis melakukan eksekusi order **Buy** atau **Sell** berdasarkan sinyal **EMA 9** dan **EMA 21**. Script ini akan membuka posisi ketika terjadi crossing antara EMA cepat (9) dan EMA lambat (21).

---

### Script MQL5 (Otomatis Eksekusi Order)
```mql5
//+------------------------------------------------------------------+
//| Auto Trading with EMA Cross Strategy                             |
//+------------------------------------------------------------------+
#property strict

// Input parameters
input int FastEMA = 9;                       // Fast EMA period
input int SlowEMA = 21;                      // Slow EMA period
input ENUM_TIMEFRAMES TimeFrame = PERIOD_M15; // Timeframe (Default: M15)
input double LotSize = 0.1;                  // Lot size for orders
input double StopLoss = 50;                  // Stop Loss in points
input double TakeProfit = 100;               // Take Profit in points
input double Slippage = 3;                   // Slippage in points

// Buffers for EMA values
double FastEMA_Buffer[];
double SlowEMA_Buffer[];

// Initialization function
int OnInit() {
    // Create indicator buffers
    SetIndexBuffer(0, FastEMA_Buffer, INDICATOR_DATA);
    SetIndexBuffer(1, SlowEMA_Buffer, INDICATOR_DATA);

    // Indicator properties
    IndicatorSetString(INDICATOR_SHORTNAME, "Auto Trading EMA Cross");
    IndicatorSetInteger(INDICATOR_DIGITS, _Digits);

    return(INIT_SUCCEEDED);
}

// Main calculation function
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[]) {
    // Calculate EMA values
    for (int i = 0; i < rates_total; i++) {
        FastEMA_Buffer[i] = iMA(NULL, TimeFrame, FastEMA, 0, MODE_EMA, PRICE_CLOSE, i);
        SlowEMA_Buffer[i] = iMA(NULL, TimeFrame, SlowEMA, 0, MODE_EMA, PRICE_CLOSE, i);
    }

    // Check for buy/sell signals
    for (int i = 1; i < rates_total; i++) {
        if (FastEMA_Buffer[i] > SlowEMA_Buffer[i] && FastEMA_Buffer[i - 1] <= SlowEMA_Buffer[i - 1]) {
            // Buy signal
            if (OrderSendIfNoPosition(OP_BUY)) {
                Print("Buy Order Executed at ", TimeToString(time[i], TIME_DATE | TIME_MINUTES));
            }
        }

        if (FastEMA_Buffer[i] < SlowEMA_Buffer[i] && FastEMA_Buffer[i - 1] >= SlowEMA_Buffer[i - 1]) {
            // Sell signal
            if (OrderSendIfNoPosition(OP_SELL)) {
                Print("Sell Order Executed at ", TimeToString(time[i], TIME_DATE | TIME_MINUTES));
            }
        }
    }

    return(rates_total);
}

// Function to send an order if no position exists
bool OrderSendIfNoPosition(int orderType) {
    // Check if there's already an open position
    if (PositionSelect(Symbol())) {
        Print("Position already exists for ", Symbol());
        return false;
    }

    // Set trade parameters
    double price = (orderType == OP_BUY) ? SymbolInfoDouble(Symbol(), SYMBOL_ASK) : SymbolInfoDouble(Symbol(), SYMBOL_BID);
    double stopLoss = (orderType == OP_BUY) ? price - StopLoss * _Point : price + StopLoss * _Point;
    double takeProfit = (orderType == OP_BUY) ? price + TakeProfit * _Point : price - TakeProfit * _Point;

    // Send the order
    trade.Buy(LotSize, Symbol(), price, Slippage, stopLoss, takeProfit);
    if (trade.ResultRetcode() == TRADE_RETCODE_DONE) {
        Print("Order successfully executed: ", (orderType == OP_BUY ? "Buy" : "Sell"));
        return true;
    } else {
        Print("Error executing order: ", trade.ResultRetcode());
        return false;
    }
}
```

---

### Cara Menggunakan Script
1. **Tambahkan Script:**
   - Salin dan tempel script di atas ke folder `Experts` MetaTrader 5 dengan nama, misalnya, `Auto_Trading_EMA_Cross.mq5`.

2. **Kompilasi Script:**
   - Buka **MetaEditor**, buka file tersebut, lalu klik tombol **Compile**.
   - Pastikan tidak ada error.

3. **Aktifkan Auto Trading:**
   - Di MetaTrader 5, klik tombol **Auto Trading** untuk mengaktifkan robot trading.

4. **Tambahkan ke Chart:**
   - Seret EA (Expert Advisor) ke chart XAU/USD dengan timeframe **M15**.

---

### Parameter Utama
- **LotSize:**
   - Ukuran lot untuk order, default-nya adalah `0.1`.
- **StopLoss:**
   - Jarak dalam poin dari harga masuk ke level stop loss, default-nya adalah `50`.
- **TakeProfit:**
   - Jarak dalam poin dari harga masuk ke level take profit, default-nya adalah `100`.
- **Slippage:**
   - Toleransi selisih harga dalam poin saat mengeksekusi order, default-nya adalah `3`.

---

### Fitur Tambahan
- **Kondisi Tidak Ada Posisi:**
   - EA akan memeriksa apakah sudah ada posisi terbuka untuk pasangan mata uang yang sama sebelum membuka order baru.
- **Eksekusi Order:**
   - Membuka posisi **Buy** saat EMA cepat (9) memotong EMA lambat (21) dari bawah.
   - Membuka posisi **Sell** saat EMA cepat (9) memotong EMA lambat (21) dari atas.
- **Stop Loss dan Take Profit:**
   - Menetapkan level Stop Loss dan Take Profit untuk melindungi modal Anda.

---

