# home-assistant-huawei-energy-managment
Code examples for Home Assistant energy management. Focus on optimizing self-consumption, reducing costs, and improving efficiency with batteries, solar, and EV charging. Hobby project – shared as-is, use at your own risk.

Min setup bygger på sensorer från Huawei Solar Integration (https://github.com/wlcrs/huawei_solar)
- Solceller
- Batteri
- Smart energimätare (Huawei)
- Laddbox (Wallbox Pulsar Max)

Steg 1
Skapa först en sensor som håller koll på husets aktuella effekt. I min setup valde jag att exkludera laddningen av elbilen. Skapa en sensor template sensor som du namger huseffekt_exl_elbil. Skapa därefter en sensor som ackumulerar kWh som du namnger energy_total_exl_elbil. 

Steg 2
Skapa SQL-sensorer för att följa din energiförbrukning. Observera att du behöver ersätta sensor.energy_total_exl_elbil om du valde något annat namn. 

Steg 3
Skapa en command line sensor för att solelsprognos. (https://forecast.solar/)
