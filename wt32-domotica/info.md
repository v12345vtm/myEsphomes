# WT32-Domotica ESPHome Configuration Manual and Technical Review

---

## Overview

This ESPHome configuration is designed for the WT32 board with native Ethernet and a DSMR P1 smart meter integration. It implements advanced logic for capacity tariff monitoring, flow rate calculation, and peak usage alerting via Home Assistant or similar platforms.

---

## 1. ⚙️ Technical Overview & Hardware

### Device Role
The device acts as a robust DSMR P1 meter reader focused on electricity capacity tariff (capaciteitstarief) management. Utilizing WT32’s Ethernet for stable networking, it uses custom time-based logic, global variables, and binary sensors for persistent state and alerts.

### Key Components

| Component | Function | Status |
| --- | --- | --- |
| DSMR | Reads smart meter data (energy, demand, voltage, gas, water) every 5 seconds | OK |
| Ethernet | LAN8720 PHY providing Ethernet connectivity | OK |
| Time (SNTP) | Syncs time using Europe/Brussels timezone for time-dependent logic | OK |
| Alerts / Logic | Custom binary sensors for peak alerts and Do Not Disturb | Advanced |

---

## 2. 🚀 Key Feature Manual

### 2.1 Capacity Tariff Management

Tracks and reacts to the monthly maximum 15-minute average demand.

| Entity Name | ESPHome ID | Description |
| --- | --- | --- |
| Capaciteitstarief huidige maand | `active_energy_import_maximum_demand_running_month` | The actual 15-minute monthly peak demand (kW) |
| Laatste Maandpiek Tijdstip | `laatste_maandpiek_tijdstip` | Timestamp (HH:MM:SS) when last monthly peak was recorded |
| Capaciteitstarief Status Text | N/A | Text status relative to current demand and maandpiek_value |
| Capaciteitstarief hoeveel %... | `capaciteitstarief_pct_until_burned` | 15-min demand as percent of peak demand or 2.5 kW minimum |

### 2.2 Time-Based Alerts and Do Not Disturb

Controls activation of alerts like SMS triggers.

| Entity Name | ESPHome ID | Type | Default | Description |
| --- | --- | --- | --- | --- |
| Nacht_start | `time_nacht_start` | datetime | 21:30:00 | Start time for "Do Not Disturb" |
| Nacht_stop | `time_nacht_stopt` | datetime | 07:45:00 | End time for "Do Not Disturb" |
| Niet Storen | `niet_storen_tussen` | binary_sensor | OFF | ON if time is between Nacht_start and Nacht_stop and 24/24h sms switch is OFF |
| 24/24h sms | `main_switch` | switch | OFF | Overrides Do Not Disturb; enables alerts 24/7 |
| VerwittigMeOpKwh | N/A | text | "3" | Configurable kW threshold to trigger peak alert |

### 2.3 Calculated Flow Rates

Uses DSMR total counters to calculate instantaneous flow rates every 60 seconds.

| Entity Name | Unit | Description |
| --- | --- | --- |
| Water Flow Rate | L/min | Real-time water consumption rate |
| Gas Flow Rate | L/min | Real-time gas consumption rate |

---

## 3. 📝 Technical Review and Recommendations

### 3.1 ⚠️ Critical Improvement: Peak Value Reset Logic

**Current:** The global `maandpiek_value` updates only when 15-minute demand drops briefly to between 0.0 and 0.1 kW, missing resets at new month start.

**Issue:** This logic depends on a narrow conditional event and does not reset the peak properly monthly.

**Recommended fix:** Add a time trigger for monthly reset at midnight on day 1:


---

## 4. How It Works

- **Globals:** The global variable `maandpiek_value` stores the current month’s peak consumption fallback at 2.5 kW.
- **Ethernet:** Controls and monitors LAN8720 Ethernet PHY for network connectivity.
- **DSMR Component:** Reads P1 smart meter data via serial UART every 5 seconds with no CRC check.
- **Time:** Uses SNTP to sync time with Europe/Brussels timezone.
- **Sensors:** Track electricity, gas, and water consumption, power demand peaks, and tariff periods.
- **Binary Sensors:** Include error detection for UART data flow, thresholds for peak alerts, and "Do Not Disturb" based on configurable night times.
- **Text Sensors:** Provide pricing, tariff, and status information.
- **Switches:** Allow toggling SMS alerts for peak usage and enabling 24/7 notification override.
- **Buttons:** Offer firmware restart triggers.

---

## 5. How to Use

1. Upload this YAML firmware to your WT32 board via ESPHome.
2. Ensure Ethernet is connected and device receives an IP address.
3. Adjust `VerwittigMeOpKwh` text input to set your peak demand alert level (default 3 kW).
4. Set tariff prices for gas, water, and electricity via respective text sensors.
5. Monitor consumption and status sensors in your Home Assistant or other integration.
6. Use the main switch to toggle 24/7 SMS alerts; use Do Not Disturb times to silence alerts overnight.
7. For best reliability, implement the recommended monthly peak reset and UART error detection improvements.

---

This comprehensive manual and review provide detailed technical insights, YAML snippet recommendations, and usage notes for your WT32-Domotica ESPHome DSMR P1 meter configuration, empowering you to manage and extend the setup reliably.


