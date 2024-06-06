# Home Energy Optimization Protocol (HEOP)

#### Übersicht

Das Energiemanagementsystem besteht aus einem zentralen Controller und verschiedenen Geräten im Hausnetz. Die Geräte sind in drei Haupttypen unterteilt und kommunizieren über das MQTT-Protokoll mit dem Controller. Der zentrale Controller, umgesetzt in Node-RED, steuert die Energieverteilung basierend auf den aktuellen Bedingungen und den Prioritäten der Geräte.

#### Gerätetypen

### Typ 1: Einfache Ein-/Ausschaltgeräte
- **Beschreibung:** Diese Geräte können ein- und ausgeschaltet werden und verbrauchen entweder eine konstante oder variable Strommenge. Sie müssen für eine bestimmte Mindestzeit eingeschaltet bleiben.
- **Parameter:**
  - `power_consumption`: Konstante oder variable Stromaufnahme (in Watt)
  - `min_on_time`: Mindestzeit, die das Gerät eingeschaltet bleiben muss (in Minuten)
  - `priority_class`: Prioritätsklasse, um eine standardisierte Priorität zu setzen
  - `max_off_time` (optional): Maximale Zeit, die das Gerät ausgeschaltet bleiben kann (in Minuten)

### Typ 2: Durchlaufgeräte
- **Beschreibung:** Diese Geräte müssen, wenn sie einmal eingeschaltet sind, ihren Zyklus komplett durchlaufen. Dazu gehören Geräte wie Waschmaschinen oder Geschirrspüler.
- **Parameter:**
  - `estimated_power`: Geschätzter Stromverbrauch für einen kompletten Zyklus (in kWh)
  - `estimated_duration`: Geschätzte Dauer des gesamten Zyklus (in Minuten)
  - `priority_class`: Prioritätsklasse, um eine standardisierte Priorität zu setzen
  - `max_off_time` (optional): Maximale Zeit, die das Gerät ausgeschaltet bleiben kann (in Minuten)

### Typ 3: Regelbare Geräte
- **Beschreibung:** Diese Geräte können nicht nur ein- und ausgeschaltet werden, sondern auch in ihrer Stromaufnahme geregelt werden. Beispiele sind elektrische Heizungen oder Klimaanlagen.
- **Parameter:**
  - `min_power`: Mindeststromaufnahme (in Watt), die das Gerät benötigt, um zu funktionieren
  - `max_power`: Maximale Stromaufnahme (in Watt), die das Gerät aufnehmen kann
  - `priority_class`: Prioritätsklasse, um eine standardisierte Priorität zu setzen
  - `min_on_time`: Mindestzeit, die das Gerät eingeschaltet bleiben muss (in Minuten)
  - `max_off_time` (optional): Maximale Zeit, die das Gerät ausgeschaltet bleiben kann (in Minuten)

## Prioritätsklassen

Geräte werden basierend auf ihrer Wichtigkeit und ihrem Energiebedarf in Prioritätsklassen eingeteilt. Die Prioritätsklassen ermöglichen eine standardisierte Steuerung durch den zentralen Controller.

**Prioritätsklassen:**
- **Kritische Geräte (Prioritätsklasse 1):** 1 - 127
- **Wichtige Geräte (Prioritätsklasse 2):** 128 - 255
- **Mittlere Wichtigkeit (Prioritätsklasse 3):** 256 - 511
- **Niedrige Wichtigkeit (Prioritätsklasse 4):** 512 - 767
- **Sehr niedrige Wichtigkeit (Prioritätsklasse 5):** 768 - 1023

## Heartbeat-Protokoll

Das Heartbeat-Protokoll übermittelt dynamische Informationen über den aktuellen Zustand des Geräts sowie optionale erwartete Stromverbrauchswerte. Eine Heartbeat-Nachricht muss alle 60 Sekunden gesendet werden. Der Controller toleriert das Fehlen von bis zu zwei Heartbeat-Nachrichten. Bei Ausbleiben der dritten Nachricht wird das Gerät gelöscht.

**Pflichtfelder:**
- `device_id`: Eindeutige Kennung des Geräts (z.B. MAC-Adresse, Hersteller und Seriennummer)
- `status`: Der aktuelle Status des Geräts (z.B. `on`, `off`, `override_on`, `override_off`, `error`)
- `current_power`: Aktuelle Stromaufnahme (in Watt)

**Optionale Felder:**
- `expected_power_1min`: Erwarteter Stromverbrauch in der nächsten Minute (in Wattstunden)
- `expected_power_10min`: Erwarteter Stromverbrauch in den nächsten 10 Minuten (in Wattstunden)
- `expected_power_1h`: Erwarteter Stromverbrauch in der nächsten Stunde (in Wattstunden)
- `expected_power_10h`: Erwarteter Stromverbrauch in den nächsten 10 Stunden (in Wattstunden)

**Beispiel-Heartbeat:**
```json
{
    "device_id": "manufacturer123-serial456",
    "status": "override_on",
    "current_power": 500,
    "expected_power_1min": 5,
    "expected_power_10min": 50,
    "expected_power_1h": 300,
    "expected_power_10h": 3000
}
```

## Eindeutige Geräte-IDs

Die `device_id` muss eindeutig sein und kann z.B. wie folgt zusammengesetzt werden:
- **MAC-Adresse:** `mac-00:1A:2B:3C:4D:5E`
- **Hersteller und Seriennummer:** `manufacturer123-serial456`

## Kommunikation über MQTT

Die Geräte kommunizieren mit dem zentralen Controller über das MQTT-Protokoll. Dies umfasst das Senden von Heartbeat-Nachrichten und die Registrierung der Geräte. Ein individuell definierbares Präfix ermöglicht eine flexible Anpassung der Themenhierarchie.

**MQTT-Topics mit Präfix:**
- **Registrierung:** `{prefix}/devices/register`
- **Heartbeat:** `{prefix}/devices/{device_id}/heartbeat`
- **Status:** `{prefix}/devices/{device_id}/status`
- **Steuerung:** `{prefix}/devices/{device_id}/control`
- **Fehler:** `{prefix}/devices/{device_id}/error`
- **Logs:** `{prefix}/devices/{device_id}/log`

**Beispiel-Präfix:** `home/energy`

## Kommunikationsablauf

1. **Registrierung des Geräts:**
   - Das Gerät sendet eine Registrierungsnachricht an `{prefix}/devices/register`.
   - Der zentrale Controller empfängt diese Nachricht und speichert die Geräteinformationen.
   - Die Registrierungsnachricht sollte als retained message gesendet werden.

2. **Regelmäßige Heartbeats:**
   - Das Gerät sendet regelmäßig (alle 60 Sekunden) Heartbeat-Nachrichten an `{prefix}/devices/{device_id}/heartbeat`, um seinen Status und erwarteten Stromverbrauch zu aktualisieren.
   - Der Controller toleriert das Fehlen von bis zu zwei Heartbeat-Nachrichten. Bei Ausbleiben der dritten Nachricht wird das Gerät als inaktiv betrachtet und aus der Liste entfernt.

3. **Statusüberwachung und Steuerung:**
   - Der Controller abonniert `{prefix}/devices/{device_id}/status`, um den Status der Geräte zu überwachen.
   - Der Controller sendet Steuerbefehle an `{prefix}/devices/{device_id}/control`, um Geräte ein- oder auszuschalten oder die Stromaufnahme zu regulieren.

4. **Fehler- und Logmeldungen:**
   - Geräte senden Fehlermeldungen an `{prefix}/devices/{device_id}/error`.
   - Geräte senden Log-Nachrichten an `{prefix}/devices/{device_id}/log`.

## Nutzung von Retained Messages

**Registrierung:**
- Die Registrierungsmeldung sollte als retained message gesendet werden, damit der Controller die letzte bekannte Gerätekonfiguration auch bei Neustart oder neuen Abonnenten sofort erhält.

## Zusammenfassung

Durch das erneute Senden der Heartbeat-Nachricht alle 60 Sekunden bleibt der Controller über den Zustand des Geräts informiert. Die Nutzung von retained messages wird nur für die Registrierungsmeldung verwendet, um die Geräteinformationen beim Neustart oder für neue Abonnenten bereitzustellen. Die Festlegung einer eindeutigen `device_id` und die Toleranz von maximal zwei fehlenden Heartbeat-Nachrichten gewährleisten die Zuverlässigkeit und Konsistenz des Systems. Die strukturierte Kommunikation im Energiemanagementsystem wird durch die MQTT-Themenhierarchie mit einem individuell definierbaren Präfix unterstützt.
