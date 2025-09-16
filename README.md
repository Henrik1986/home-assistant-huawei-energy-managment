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

Steg 1: Skapa en template (se template.yaml) som håller koll på husets aktuella effekt. I min setup valde jag att exkludera laddningen av elbilen  - ge den namnet: huseffekt_exl_elbil. Skapa därefter en sensor (se sensors.yaml) som ackumulerar kWh som du namnger energy_total_exl_elbil. 
Min kod förutsätter att du har en smartmätare via Huawei intergrationen samt att du justerat dayily_yail enligt följande: https://github.com/wlcrs/huawei_solar/wiki/Daily-Solar-Yield#a-better-approach
Om din setup ser ut på annat sätt kan du utgå från koden (se template.yaml) för att skapa en sensor som håller koll på husets aktuella effekt. 

Steg 2: Skapa SQL-sensorer (se sql-sensor) för att följa din energiförbrukning. Mina SQL-sensorer beräknar kWh/h utifrån din förbrukning de senaste 3 dagarna. Detta värde går att justera i koden. Du skapar SQL-sensorer via integrationer. (https://www.home-assistant.io/integrations/sql/)
Ange följande + frågan som finns i filerna i mappen sql-sensor
- Kolumn: avg_kwh_per_hour
- Måttenhet: kWh/h

Observera att du behöver ersätta sensor.energy_total_exl_elbil om du valde något annat namn i steg 1. 

Steg 3: Skapa en command line sensor (se command_line.yaml) för solelsprognos (data hämtas från https://forecast.solar/)
Observer att du behöver ersätta sensor.solar_forecast_west i resten av koden om du väljer något annat namn. Mitt råd är att använda ett annat friendly name om du önskar ett annat namn. Det kommer inte påverka kommande kod. 

Steg 4: Skapa tre template sensorer (se template.yaml) som söker upp billiga laddningsfönster (observera att template sensorerna ska ha en egen trigger). Logiken bakom laddningsfönsterna kommer du kunna styra vi UI. Du behöver även lägga in din Nordpool sensor i template koden (min är i SEK/kWh). Template sensorerna har nedanstående namn i filen template.yaml.
- battery_charge_window_cheapest_1a
- battery_charge_window_cheapest_1b
- battery_charge_window_cheapest_2

Steg 5: Skapa två input_number (via helper). Ge dem följande namn:
- battery_charge_duration_hours
- battery_total_capacity_kwh

Steg 6: Skapa en input_button (via helper) för uppdateringen av laddningsfönster. Knappen bidrar till smidig manuell uppdatering, men även för den automatiska uppdateringen. Ge den följande namn: 
- update_battery_cheapest_charge

Steg 7: Skapa en template sensor (se template.yaml) som håller koll på om nya elpriser finns tillgängliga. Observera att denna ska ligga som en binary_sensor under template. Lägg in din Nordpool-sensor i koden. 

Steg 8: Skapa en automation (battery_update_charge_interval.yaml i mappen automations) som styr uppdateringen av laddningsfönsterna. När du gjort detta har du tre laddningsfönster som uppdateras när nya elpriser släpps och som ska användas för att ladda batteriet. Sensorernas namn hittar du i steg 4. 

Steg 9: Skapa två template sensorer (se template.yaml) som beräknar energibehovet. Observera att koden är lång och att dessa sensorer inte ska ha en egen trigger. Template sensorerna har nedanstående namn i filen template.yaml. 
- battery_charge_energy_1a_2
- battery_charge_energy_2_1b

För att sensorerna ska fungera krävs sensor.batteries_state_of_capacity som visar aktuell batterinivå (finns i Huawei integrationen). Finns inte denna behöver du justera koden eller din sensors namn. Du behöver även number.batteries_end_of_discharge_soc som visar lägsta urladdninvsnivå i %. Även den följer med Huawei integrationen. Finns inte denna behöver du justera koden eller din sensors namn.

Steg 10: Skapa en input_number (via helper) för att addera kWh till energibehovet vilket skapar en möjlighet att trimma in logiken utifrån ditt energibehov. Ge sensorn följande namn: 
- battery_buffer_kwy

Steg 11: Skapa två automationer som kommer att styra om batteriet ska laddas eller inte. I mappen automations heter dessa 
- battery_luna_2000_S1_interval_1a_2.yaml
- battery_luna_2000_S1_interval_2_1b.yaml

Automationerna är byggda utifrån Huawei integrationen val och utifrån batteriet Luna2000 - S1. Om du har en annan setup får du justera automationerna utifrån den. Har du samma setup behöver du bara lägga till ditt device_id. 
Detta hittar du under utvecklarverktyg -> åtgärder -> klicka på åtgärd -> skriv huawei -> välj forcible charge -> klicka på gå till UI-läge -> Välj Batteries i rullgardinen på raden battery -> Klicka på gå till YAML-läge
Då ska du få fram till device_id



Med en apex-chart kan du på ett visuellt tydligt sätt synliggöra laddningsfönster, prognos för solel och elpriser. Exmpelkod finns under Apex-chart. 
