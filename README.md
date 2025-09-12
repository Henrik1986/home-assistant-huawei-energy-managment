# home-assistant-huawei-energy-managment
Code examples for Home Assistant energy management. Focus on optimizing self-consumption, reducing costs, and improving efficiency with batteries, solar, and EV charging. Hobby project – shared as-is, use at your own risk.




Steg 1 - Ditt energibehov
Först behöver du skapa en sensor som håller koll på husets aktuella effekt. I min setup har jag valt att exkludera elbilen. Skapa en sensor template sensor som du namger huseffekt_exl_elbil. Skapa därefter en sensor som ackumulerar totalt kWh som du namnger energy_total_exl_elbil

Steg 2
Skapa SQL-sensorer för att följa din energiförbrukning. I min setup har jag delat in dygnet i o
