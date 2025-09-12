# Home Assistant Huawei Energy Managment
Code examples for Home Assistant energy management. Focus on optimizing self-consumption, reducing costs, and improving efficiency with batteries, solar, and EV charging. Hobby project – shared as-is, use at your own risk.

Min setup bygger på sensorer från Huawei Solar Integration (https://github.com/wlcrs/huawei_solar)
- Solceller
- Batteri
- Smart energimätare (Huawei)
- Laddbox (Wallbox Pulsar Max)

Steg 1: Skapa en template som håller koll på husets aktuella effekt. I min setup valde jag att exkludera laddningen av elbilen ge den namnet: huseffekt_exl_elbil. Skapa därefter en sensor som ackumulerar kWh som du namnger energy_total_exl_elbil. 

Steg 2: Skapa SQL-sensorer för att följa din energiförbrukning. 
Observera att du behöver ersätta sensor.energy_total_exl_elbil om du valde något annat namn. 

Steg 3: Skapa en command line sensor för solelsprognos (https://forecast.solar/)
Observer att du behöver ersätta sensor.solar_forecast_west i resten av koden om du väljer något annat namn.

Steg 4: Skapa template sensorer som söker upp laddningsfönster (observera att template sensorerna ska ha en trigger). Logiken bakom laddningsfönsterna går att styra vi UI. 
Skapa en input_number 
- battery_charge_duration_hours

Du behöver även lägga in din Nordpool sensor i template koden (min är i SEK/kWh). 

Steg 5: Skapa en input_button för uppdateringen av nya laddningsfönster. 
- update_battery_cheapest_charge

Steg 6: Skapa en template som håller koll på om nya elpriser finns tillgängliga. Observera att denna ska ligga som en binary_sensor. Ersätt med din Nordpool-sensor.

Steg 7: Skapa en automation (battery_update_charge_interval.yaml) som styr uppdateringen av nya laddningsfönster
