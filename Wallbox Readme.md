Wallbox Automation (go-e Charger + PV-Überschuss)
Überblick
Diese Automation steuert eine go-e Wallbox in Home Assistant und kombiniert:
manuelle Moduswahl
PV-Überschuss-Ladung
Batterieschutz (Hausbatterie wird priorisiert)
automatische Phasenwahl (1-phasig bei PV-Laden)
Ladelogik für Sofort- und Zeitladen
Die Steuerung erfolgt über die Auswahlbox:
Code kopieren

input_select.wallbox_auswahl
mit folgenden Optionen:
Aus
Überschuss PV
Laden sofort (11kW)
Laden Zeit (11kW)
Funktionsprinzip
1. Nach Abstecken → automatisch auf PV-Überschuss
Sobald das Fahrzeug abgesteckt wird, schaltet die Automation die Auswahl automatisch zurück auf:
Überschuss PV
Ziel: Standardmäßig sanft und PV-geführt laden.
2. PV-Session-Merker
Die Automation nutzt einen Merker:
Code kopieren

input_boolean.wallbox_pv_session
Er bedeutet:
„PV-Überschuss-Laden läuft bereits.“
Ab diesem Zeitpunkt wird der Hausbatterie-SOC ignoriert, um die Ladung nicht unnötig zu unterbrechen.
3. Startbedingungen für PV-Überschuss
PV-Ladung startet nur, wenn:
PV-Leistung > 800 W, und
Hausbatterie-SOC ≥ 85 %,
oder die PV-Session läuft bereits.
Zusätzlich:
Bei SOC < 75 % wird der Start blockiert (Priorität Haus).
4. 1-phasiges Laden bei PV-Überschuss
Über:
Code kopieren

select.goe_259381_psm
wird automatisch gesetzt:
Modus
Phase(n)
Wert
Überschuss PV
1-phasig
"1"
alle anderen Modi
Auto
"0"
Laden aus
Auto
"0"
5. Weitere Modi
Laden sofort (11 kW)
16 A, Auto-Phasenmodus
keine SOC- oder PV-Bedingung
Laden Zeit (11 kW)
16 A
nur im definierten Zeitfenster
input_boolean.wallbox_boost muss aktiv sein
6. Beenden der PV-Session
Die PV-Session endet, wenn:
Auto abgesteckt
Ladestrom = 0
Modus ≠ „Überschuss PV“
Entitätenübersicht
Steuerung
Zweck
Entity
Moduswahl
input_select.wallbox_auswahl
PV-Session
input_boolean.wallbox_pv_session
Boost
input_boolean.wallbox_boost
Zeit Start
input_datetime.wallbox_zeit_start
Zeit Ende
input_datetime.wallbox_zeit_ende
Sensoren / Messwerte
Zweck
Entity
Hausbatterie SOC
sensor.victron_battery_soc
PV-Leistung gesamt (W)
number.pv_gesamt
Auto SOC
sensor.skoda_elroq_batteriestand
Ladelimit Auto
number.skoda_elroq_ladelimit
Auto verbunden
binary_sensor.goe_259381_car_0
Wallbox-Parameter
Zweck
Entity
Freigabe (FRC)
select.goe_259381_frc
Soll-Ampere
number.goe_259381_amp
Phasenmodus
select.goe_259381_psm
Soll-Ampere-Helper
input_number.wallbox_ladestrom
Sicherheits- und Designprinzipien
Kein Laden bei leerem Hausakku (außer expliziten Modi)
Session-Merker verhindert Flattern
1-phasig im PV-Modus, um PV besser auszunutzen
Failsafe: Laden wird gesperrt, wenn Bedingungen fehlen
Debugging
Es existiert zusätzlich ein Markdown-Debug-Panel mit:
Modus
PV-Session-Status
SOC Haus / Auto
PV-Leistung
Ladestrom Soll/Ist
Phasen- und Freigabestatus
Damit lassen sich Entscheidungen transparent nachvollziehen.
Weiterentwicklungsideen
dynamische Mindest-PV-Schwelle (statt Fixwert 800 W)
Anpassung der Ampere-Limits an Fahrzeug / Stromanschluss
Logging jeder Ladesession (Start/Stop, geladene kWh, Gründe)
Lizenz
Frei nutzbar, solange du weißt, was du tust
und dir bewusst bist:
Lade- und Netzsicherheit bleiben immer deine Verantwortung.
Wenn du möchtest, können wir:
eine strukturierte GitHub-Ordnerstruktur vorschlagen (automation, helpers, docs), oder
ein kleines Diagramm ergänzen, das den Entscheidungsfluss visualisiert.
