# Home Assistant Huawei Energy Managment
Code examples for Home Assistant energy management. Focus on optimizing self-consumption, reducing costs, and improving efficiency with batteries, solar, and EV charging. Hobby project – shared as-is, use at your own risk.

Min setup

Huawei Solar Integration (https://github.com/wlcrs/huawei_solar)
- Solceller
- Batteri
- Smart energimätare

Laddbox (https://www.home-assistant.io/integrations/wallbox/)
- Wallbox Pulsar Max 

<img width="1812" height="814" alt="Skärmbild 2025-09-15 221922" src="https://github.com/user-attachments/assets/36321474-aa4d-4384-b6cf-5126dd4b973d" /> Bilden ovan är en skärmbild från min vy i Home Assistant där jag justera logiken. 

Steg 1: Skapa en template som håller koll på husets aktuella effekt. I min setup valde jag att exkludera laddningen av elbilen  - ge den namnet: huseffekt_exl_elbil. Skapa därefter en sensor som ackumulerar kWh som du namnger energy_total_exl_elbil. 

Steg 2: Skapa SQL-sensorer (integration) för att följa din energiförbrukning. 
- Kolumn: avg_kwh_per_hour
- Måttenhet: kWh/h

Observera att du behöver ersätta sensor.energy_total_exl_elbil om du valde något annat namn i steg 1. 

Steg 3: Skapa en command line sensor för solelsprognos (https://forecast.solar/)
Observer att du behöver ersätta sensor.solar_forecast_west i resten av koden om du väljer något annat namn.

Steg 4: Skapa template sensorer som söker upp laddningsfönster (observera att template sensorerna ska ha en trigger). Logiken bakom laddningsfönsterna går att styra vi UI. 
Skapa en input_number 
- battery_charge_duration_hours

Du behöver även lägga in din Nordpool sensor i template koden (min är i SEK/kWh). 

Steg 5: Skapa en input_button för uppdateringen av laddningsfönster. 
- update_battery_cheapest_charge

Steg 6: Skapa en template som håller koll på om nya elpriser finns tillgängliga. Observera att denna ska ligga som en binary_sensor. Ersätt med din Nordpool-sensor.

Steg 7: Skapa en automation (battery_update_charge_interval) som styr uppdateringen av laddningsfönsterna. Nu har du tre laddningsfönster som uppdateras när nya elpriser släpps och som ska användas för att ladda batteriet
- battery_charge_window_cheapest_1a
- battery_charge_window_cheapest_1b
- battery_charge_window_cheapest_2
