# Home Assistant Huawei Energy Managment
Code examples for Home Assistant energy management. Focus on optimizing self-consumption, reducing costs, and improving efficiency with batteries, solar, and EV charging. 

> [!WARNING]
> Hobby project – shared as-is, use at your own risk.

> [!IMPORTANT]
> You have du adjust the code to your setup.

> [!NOTE]
> The system is currently being tested with 15-minute electricity price intervals.

### Introduction 
The smart charging logic search for cheap charging intervals when new electricity prices are published. Then it calculates how much energy is needed between the charging intervalls. A solar forecast is included to increase self-sufficiency. If solar power isn’t enough and the price is very low (set in the UI), the battery charges extra from the grid. If prices are high now but low at the next charging window, any surplus will be sold.

💥 BONUS! 
- Battery discharge can be limited to only cover the home’s demand, which is useful when charging an EV (battery_luna_2000_S1_ev_protection.yaml in dictionary automations)
- Automation that limits solar power export when the electricity price is zero or lower (battery_luna_2000_S1_solar_export.yaml in dictionary automations)

<img width="1850" height="783" alt="Skärmbild 2025-10-04 154057" src="https://github.com/user-attachments/assets/c10d29ba-81ae-4afb-9aa9-de6e968bd0a6" />

Bilden ovan visar adminvyn via Home Assistant


## My setup 

[Huawei Solar Integration](https://github.com/wlcrs/huawei_solar)
- PV
- Battery (Luna 2000 S1)
- Smart Power Meter

[Wallbox](https://www.home-assistant.io/integrations/wallbox/)
- Wallbox Pulsar Max 

### Steg 1
Skapa en template (finns i template.yaml) som håller koll på husets aktuella effekt. I min setup valde jag att exkludera laddningen av elbilen. Ge den följande namn. 

- huseffekt_exl_elbil

Skapa därefter en sensor (finns i sensors.yaml) som ackumulerar kWh som du namnger 

- energy_total_exl_elbil. 

> [!IMPORTANT]
> Min kod förutsätter att du har en smartmätare via Huawei intergrationen samt att du justerat dayily_yail enligt [följande](https://github.com/wlcrs/huawei_solar/wiki/Daily-Solar-Yield#a-better-approach)

> [!NOTE]
> Om din setup ser ut på annat sätt kan du utgå från koden (se template.yaml) för att skapa en sensor som håller koll på husets aktuella effekt. 

### Steg 2
Skapa SQL-sensorer (finns i mappen sql-sensor) för att följa din energiförbrukning. SQL-sensorer beräknar kWh/h för de senaste 3 dagarna. Antalet dagar går att justera i koden. Skapar SQL-sensorer via [integrationer](https://www.home-assistant.io/integrations/sql/)
Ange följande + frågan som finns i filerna i mappen sql-sensor
- Kolumn: avg_kwh_per_hour
- Måttenhet: kWh/h

> [!IMPORTANT]
> Ersätt sensor.energy_total_exl_elbil om du valde något annat namn i steg 1. 

### Steg 3
Skapa en command line sensor (finns i command_line.yaml) för [solelsprognos](https://forecast.solar/)

> [!IMPORTANT]
> Ersätt sensor.solar_forecast_west i resten av koden om något annat namn valdes. Mitt råd är att använda ett annat friendly name om du önskar ett annat namn. Detta kommer inte påverka resterande kod

### Steg 4
Skapa tre template sensorer (finns i template.yaml) som söker upp billiga laddningsfönster. Template sensorerna har nedanstående namn i filen template.yaml.

- battery_charge_window_cheapest_1a
- battery_charge_window_cheapest_1b
- battery_charge_window_cheapest_2

> [!IMPORTANT]
> Template sensorerna ska ha en egen trigger och lägg in din Nordpool sensor i template koden (min är i SEK/kWh).

### Steg 5 
Skapa tre input_number (via helper). Ge dem följande värden:
- battery_charge_duration_hours
  - Lägsta värde: 1
  - Högsta värde: 6
  - Visningsläge: Inmatningsfält
  - Steglängd: 1
  - Måttenhet: h
    
- battery_total_capacity_kwh
  - Lägsta värde: 0
  - Högsta värde 100
  - Visningsläge: Inmatningsfält
  - Steglängd: 0,1
  - Måttenhet: kWh
    
- price_limit_supercheap
  - Lägsta värde: 0
  - Högsta värde: 5
  - Visningsläge: Inmatningsfält
  - Steglängd: 0,01
  - Måttenhet: SEK/kWh

### Steg 6 
Skapa en input_button (via helper) för uppdateringen av laddningsfönster. Knappen bidrar till smidig manuell uppdatering vid behov, men även för den automatiska uppdateringen. Ge den följande namn: 

- update_battery_cheapest_charge

### Steg 7
Skapa en template sensor (finns i template.yaml) som håller koll på om nya elpriser finns tillgängliga. 

> [!IMPORTANT]
> Template sensorn ska ligger som en binary_sensor under template och lägg till din Nordpool-sensor i koden.

### Steg 8 
Skapa en automation (battery_update_charge_interval.yaml i mappen automations) som styr uppdateringen av laddningsfönsterna. När du gjort detta har du tre laddningsfönster som uppdateras när nya elpriser släpps och som kommer att användas för att ladda batteriet. Sensorernas namn hittar du i steg 4. 

### Steg 9
Skapa två template sensorer (finns i template.yaml) som beräknar energibehovet. Template sensorerna har nedanstående namn i filen template.yaml. 

- battery_charge_energy_1a_2
- battery_charge_energy_2_1b

> [!IMPORTANT]
> Observera att koden är lång och att dessa sensorer INTE ska ha en egen trigger.

> [!WARNING]
> För att sensorerna ska fungera krävs sensor.batteries_state_of_capacity som visar aktuell batterinivå. Du behöver även number.batteries_end_of_discharge_soc som visar lägsta urladdninvsnivå i %. Båda entiterna följer med [Huawei Solar Integration](https://github.com/wlcrs/huawei_solar). Finns inte dessa behöver du justera koden eller din sensorers namn.

### Steg 10
Skapa en input_number (via helper) för att addera kWh till energibehovet vilket skapar en möjlighet att trimma in logiken utifrån ditt energibehov. Ge sensorn följande värden: 

- battery_buffer_kwh
   - Lägsta värde: 0
   - Högsta värde: 10
   - Visningsläge: Inmatningsfält
   - Steglängd: 1
   - Måttenhet: kWh

### Steg 11
Skapa två automationer som kommer att styra om batteriet ska laddas eller inte. I mappen automations heter dessa 

- battery_luna_2000_S1_interval_1a_2.yaml
- battery_luna_2000_S1_interval_2_1b.yaml

> [!WARNING]
>Automationerna är byggda utifrån [Huawei Solar Integration](https://github.com/wlcrs/huawei_solar) och utifrån batteriet Luna2000 - S1. Om du har en annan setup får du justera automationerna utifrån den. Har du samma setup behöver du bara lägga till ditt device_id. Detta hittar på följande sätt:
> 
> Utvecklarverktyg -> Åtgärder -> Klicka på åtgärd -> Skriv huawei -> Välj forcible charge -> Klicka på gå till UI-läge -> Välj Batteries i rullgardinen på raden battery -> Klicka på gå till YAML-läge - Då ska du få fram till device_id

### Steg 12
Skapa två input_boolean (via helper) för att inte laddningsautomationerna ska påverka varandra. Ge dem följande namn: 
- interval_1a
- interval_2
  
### Steg 13
Skapa en input_boolean (via helper) som avaktiverar samtliga automationer för att kunna ladda manuelt. Ge den med följande namn: 
- manual_charge

### Steg 14
Skapa en automation som kommer att styra om batteriet ska sälja om elpriser är högt och kommande elpris är lågt. I mappen automations heter denna
- battery_luna_2000_S1_discharge.yaml

> [!IMPORTANT]
> Lägg in din Nordpool-sensor

### Steg 15
Skapa två input_number (via helper) för att kunna justera gränsvärdena för att sälja överskottet. Ge input_number förljande värden. 

- battery_charge_price
    - Lägsta värde: 0
    - Högsta värde: 5
    - Visningsläge: Inmatningsfält
    - Steglängd: 0,01
    - Måttenhet: SEK/kWH
      
- expensive_electricity
    - Lägsta värde: 0
    - Högsta värde: 10
    - Visningsläge: Inmatningsfält
    - Stegläng: 0,1
    - Måttenhete SEK/kWH

### Steg 16
Skapa en ny vy i Home Assistant och lägg in koden från filen admin_view.yaml. Via den nya vyn kan du nu justera värdena som styr laddningslogiken men även följa hur laddningslogiken arbetar. 

> [!IMPORTANT]
> Klistra in koden genom att klicka på "pennan" som finns brevid din nya vy i "rubriken". Välj därefter redigera i YAML. Klistra sen in hela koden där.

> [!NOTE]
> Du behöver installera följande custom intergration via HACS - Apexcharts-card, Layout-card, Stack-In-Card.

> [!NOTE]
> Du behöver också installera SMHI tillägget och konfiguerar det utifrån din position 

> [!NOTE]
> Om något kort inte fungerar beror det troligtvis på att rätt entitet inte valts. 
