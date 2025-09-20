# Home Assistant Huawei Energy Managment
Code examples for Home Assistant energy management. Focus on optimizing self-consumption, reducing costs, and improving efficiency with batteries, solar, and EV charging. 

> [!WARNING]
> Hobby project – shared as-is, use at your own risk.

> [!IMPORTANT]
> You have du adjust the code to your setup.

### My setup 

[Huawei Solar Integration](https://github.com/wlcrs/huawei_solar)
- PV
- Battery (Luna 2000 S1)
- Smart Power Meter

[Wallbox](https://www.home-assistant.io/integrations/wallbox/)
- Wallbox Pulsar Max 

### Steg 1
Skapa en template (se template.yaml) som håller koll på husets aktuella effekt. I min setup valde jag att exkludera laddningen av elbilen  - ge den namnet: huseffekt_exl_elbil. Skapa därefter en sensor (se sensors.yaml) som ackumulerar kWh som du namnger energy_total_exl_elbil. 
Min kod förutsätter att du har en smartmätare via Huawei intergrationen samt att du justerat dayily_yail enligt [följande](https://github.com/wlcrs/huawei_solar/wiki/Daily-Solar-Yield#a-better-approach)

> [!IMPORTANT]
> Om din setup ser ut på annat sätt kan du utgå från koden (se template.yaml) för att skapa en sensor som håller koll på husets aktuella effekt. 

### Steg 2
Skapa SQL-sensorer (se mappen sql-sensor) för att följa din energiförbrukning. SQL-sensorer beräknar kWh/h för de senaste 3 dagarna. Antalet dagar går att justera i koden. Skapar SQL-sensorer via [integrationer](https://www.home-assistant.io/integrations/sql/)
Ange följande + frågan som finns i filerna i mappen sql-sensor
- Kolumn: avg_kwh_per_hour
- Måttenhet: kWh/h

> [!IMPORTANT]
> Ersätt sensor.energy_total_exl_elbil om du valde något annat namn i steg 1. 

### Steg 3
Skapa en command line sensor (se command_line.yaml) för [solelsprognos](https://forecast.solar/)

> [!IMPORTANT]
> Ersätt sensor.solar_forecast_west i resten av koden om något annat namn valdes. Mitt råd är att använda ett annat friendly name om du önskar ett annat namn. Detta kommer inte påverka resterande kod

### Steg 4
Skapa tre template sensorer (se template.yaml) som söker upp billiga laddningsfönster. Template sensorerna har nedanstående namn i filen template.yaml.
- battery_charge_window_cheapest_1a
- battery_charge_window_cheapest_1b
- battery_charge_window_cheapest_2

> [!IMPORTANT]
> Template sensorerna ska ha en egen trigger och lägg in din Nordpool sensor i template koden (min är i SEK/kWh).

### Steg 5 
Skapa tre input_number (via helper). Ge dem följande namn:
- battery_charge_duration_hours
  - Lägsta värde: 1
  - Högsta värde: 6
  - Steglängd: 1
  - Måttenhet: h
- battery_total_capacity_kwh
  - Lägsta värde: 0
  - Högsta värde 100
  - Steglängd: 0,1
  - Måttenhet: kWh
- price_limit_supercheap
  - Lägsta värde: 0
  - Högsta värde: 5
  - Steglängd: 0,01
  - Måttenhet: SEK/kWh

### Steg 6 
Skapa en input_button (via helper) för uppdateringen av laddningsfönster. Knappen bidrar till smidig manuell uppdatering vid behov, men även för den automatiska uppdateringen. Ge den följande namn: 
- update_battery_cheapest_charge

### Steg 7
Skapa en template sensor (se template.yaml) som håller koll på om nya elpriser finns tillgängliga. 

> [!IMPORTANT]
> Template sensorn ska ligger som en binary_sensor under template och lägg till din Nordpool-sensor i koden.

### Steg 8 
Skapa en automation (battery_update_charge_interval.yaml i mappen automations) som styr uppdateringen av laddningsfönsterna. När du gjort detta har du tre laddningsfönster som uppdateras när nya elpriser släpps och som kommer att användas för att ladda batteriet. Sensorernas namn hittar du i steg 4. 

### Steg 9
Skapa två template sensorer (se template.yaml) som beräknar energibehovet. Template sensorerna har nedanstående namn i filen template.yaml. 
- battery_charge_energy_1a_2
- battery_charge_energy_2_1b

> [!IMPORTANT]
> Observera att koden är lång och att dessa sensorer inte ska ha en egen trigger.

> [!WARNING]
> För att sensorerna ska fungera krävs sensor.batteries_state_of_capacity som visar aktuell batterinivå. Du behöver även number.batteries_end_of_discharge_soc som visar lägsta urladdninvsnivå i %. Båda entiterna följer med [Huawei Solar Integration](https://github.com/wlcrs/huawei_solar). Finns inte dessa behöver du justera koden eller din sensorers namn.

### Steg 10
Skapa en input_number (via helper) för att addera kWh till energibehovet vilket skapar en möjlighet att trimma in logiken utifrån ditt energibehov. Ge sensorn följande namn: 
- battery_buffer_kwh
   - Lägsta värde: 0
   - Högsta värde: 10
   - Steglängd: 1
   - Måttenhet: kWh

### Steg 11
Skapa två automationer som kommer att styra om batteriet ska laddas eller inte. I mappen automations heter dessa 
- battery_luna_2000_S1_interval_1a_2.yaml
- battery_luna_2000_S1_interval_2_1b.yaml

> [!WARNING]
>Automationerna är byggda utifrån [Huawei Solar Integration](https://github.com/wlcrs/huawei_solar) och utifrån batteriet Luna2000 - S1. Om du har en annan setup får du justera automationerna utifrån den. Har du samma setup behöver du bara lägga till ditt device_id. 
Detta hittar på följande sätt: Utvecklarverktyg -> Åtgärder -> Klicka på åtgärd -> Skriv huawei -> Välj forcible charge -> Klicka på gå till UI-läge -> Välj Batteries i rullgardinen på raden battery -> Klicka på gå till YAML-läge - Då ska du få fram till device_id

### Steg 12
Skapa en input_boolean (via helper) som avaktiverar samtliga automationer för att kunna ladda nu. Namge ge den med följande namn: 
- manual_charge

### Steg 13
Skapa en ny vy i Home Assistant och lägg in koden från filen admin_view.yaml. Via den nya vyn kan du nu justera värdena som styr laddningslogiken men även följa hur laddningslogiken arbetar. 

