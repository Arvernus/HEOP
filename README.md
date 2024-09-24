# Home Energy Optimization Protocol (HEOP)

## Übersicht

Das **Home Energy Optimization Protocol (HEOP)** ist ein spezifiziertes Kommunikationsprotokoll zur effizienten Steuerung und Optimierung der Energieverteilung und des Energieverbrauchs in Smart Homes. Es ermöglicht die Integration und Koordination verschiedener elektrischer Geräte und Systeme innerhalb eines Haushalts, um den Energieverbrauch zu minimieren, die Nutzung erneuerbarer Energien zu maximieren und die Betriebskosten zu senken.

HEOP verwendet das **MQTT 5.0** Protokoll zur Kommunikation zwischen einem oder mehreren zentralen **Controllern** und den verbundenen Geräten im Hausnetz. Die Controller überwachen den Status und den Energieverbrauch der Geräte, senden Steuerbefehle und stellen sicher, dass die Energienutzung basierend auf vordefinierten Prioritäten und Parametern optimiert wird.

### Hauptfunktionen von HEOP

- **Echtzeit-Überwachung und -Steuerung**
- **Priorisierung von Geräten**
- **Energieoptimierung**
- **Heartbeat-Protokoll**
- **Fehlermanagement**
- **Datenprotokollierung und Datenschutz**
- **Sicherheitsaspekte**
- **Mehrere Controller und Redundanz**

## Gerätetypen

*(Gerätetypen und deren Parameter bleiben unverändert und sind bereits präzise definiert.)*

## Prioritätsklassen

*(Prioritätsklassen bleiben unverändert.)*

## Heartbeat-Protokoll

*(Das Heartbeat-Protokoll bleibt unverändert und enthält bereits genaue Felddefinitionen.)*

## Eindeutige Geräte-IDs

Die `device_id` muss eindeutig sein und wird als UUID v4 (Universally Unique Identifier Version 4) verwendet. Dies stellt sicher, dass jede Geräte-ID universell eindeutig ist.

**Beispiel:**

- `device_id`: `550e8400-e29b-41d4-a716-446655440000`

## Kommunikation über MQTT

Die Kommunikation erfolgt über **MQTT Version 5.0**. Die folgenden Spezifikationen sind zu beachten:

- **Transport Layer Security (TLS):**
  - Es ist **TLS 1.3** zu verwenden.
  - Die erlaubten Cipher Suites sind die in RFC 8446 definierten sicheren Cipher Suites, z. B. `TLS_AES_128_GCM_SHA256`.

- **Authentifizierung:**
  - **Mutual TLS (mTLS)** wird für die gegenseitige Authentifizierung zwischen Geräten und Controllern eingesetzt.
  - Jedes Gerät und jeder Controller muss ein gültiges X.509 Zertifikat besitzen.
  - Die Zertifikate müssen von einer vertrauenswürdigen internen Zertifizierungsstelle (CA) ausgestellt werden.

- **Autorisierung:**
  - MQTT Access Control Lists (ACLs) werden verwendet, um den Zugriff auf bestimmte Topics zu steuern.
  - Geräte dürfen nur auf ihre eigenen Topics zugreifen.
  - Controller haben Lesezugriff auf alle Gerätetopics und Schreibzugriff auf Kontrolltopics.

**MQTT-Topics mit Präfix:**

- **Registrierung:** `{prefix}/devices/register`
- **Heartbeat:** `{prefix}/devices/{device_id}/heartbeat`
- **Status:** `{prefix}/devices/{device_id}/status`
- **Steuerung:** `{prefix}/devices/{device_id}/control`
- **Fehler:** `{prefix}/devices/{device_id}/error`
- **Logs:** `{prefix}/devices/{device_id}/log`

**Nachrichtenformat:**

- Alle Nachrichten müssen im **JSON-Format** kodiert sein.
- Die JSON-Struktur muss den definierten Feldern und Datentypen entsprechen.

## Kommunikationsablauf

1. **Registrierung des Geräts:**

   - Das Gerät sendet eine Registrierungsnachricht an `{prefix}/devices/register`.
   - **Nachrichtenstruktur:**

     ```json
     {
       "device_id": "550e8400-e29b-41d4-a716-446655440000",
       "device_type": "type1",
       "parameters": {
         "power_consumption": 500,
         "min_on_time": 30,
         "priority_class": 256,
         "allowed_time_frames": ["08:00-12:00", "14:00-18:00"],
         "preferred_energy_sources": ["solar", "battery"]
       }
     }
     ```

   - Die Registrierungsnachricht muss als **Retained Message** gesendet werden.

2. **Regelmäßige Heartbeats:**

   - Das Gerät sendet alle 60 Sekunden eine Heartbeat-Nachricht an `{prefix}/devices/{device_id}/heartbeat`.
   - **Nachrichtenstruktur:**

     ```json
     {
       "device_id": "550e8400-e29b-41d4-a716-446655440000",
       "status": "on",
       "current_power": 500,
       "current_demand": 700,
       "expected_power_1min": 5,
       "expected_power_10min": 50,
       "expected_power_1h": 300,
       "expected_power_10h": 3000
     }
     ```

3. **Statusüberwachung und Steuerung:**

   - **Steuerbefehle:** Der Controller sendet Steuerbefehle an `{prefix}/devices/{device_id}/control`.
   - **Nachrichtenstruktur für Steuerbefehle:**

     ```json
     {
       "command": "set_status",
       "value": "off"
     }
     ```

     Mögliche `command` Werte:

     - `set_status`: Ändert den Status des Geräts (`on`, `off`, `standby`).
     - `set_power`: Setzt die Stromaufnahme für regelbare Geräte (Wert in Watt).

4. **Fehler- und Logmeldungen:**

   - Geräte senden Fehlermeldungen an `{prefix}/devices/{device_id}/error`.
   - **Nachrichtenstruktur für Fehlermeldungen:**

     ```json
     {
       "device_id": "550e8400-e29b-41d4-a716-446655440000",
       "error_code": "E002",
       "error_message": "Hardwarefehler: Sensor defekt",
       "timestamp": "2023-10-01T12:34:56Z"
     }
     ```

## Sicherheitsaspekte

### Transportverschlüsselung

- **TLS Version:** Es muss **TLS 1.3** gemäß [RFC 8446](https://tools.ietf.org/html/rfc8446) verwendet werden.
- **Erlaubte Cipher Suites:** Nur die folgenden Cipher Suites sind erlaubt:
  - `TLS_AES_256_GCM_SHA384`
  - `TLS_CHACHA20_POLY1305_SHA256`
  - `TLS_AES_128_GCM_SHA256`

### Authentifizierung

- **Mutual TLS (mTLS):** Beide Kommunikationspartner (Gerät und Controller) müssen sich gegenseitig mittels X.509 Zertifikaten authentifizieren.
- **Zertifikate:**
  - Zertifikate müssen den **X.509 v3** Standard erfüllen.
  - Die Schlüssellänge für RSA muss mindestens **2048 Bit** betragen.
  - Alternativ können elliptische Kurven wie **secp384r1** verwendet werden.
- **Zertifikatsverwaltung:**
  - Eine interne **Public Key Infrastructure (PKI)** wird eingesetzt.
  - Zertifikate werden von einer vertrauenswürdigen internen CA ausgestellt.
  - Zertifikate müssen regelmäßig erneuert werden (empfohlen alle 2 Jahre).

### Autorisierung

- **Access Control Lists (ACLs):**
  - Geräte dürfen nur auf ihre eigenen MQTT-Topics zugreifen.
  - Controller haben Lesezugriff auf alle Gerätetopics und Schreibzugriff auf Kontrolltopics.
  - Die ACLs müssen auf dem MQTT-Broker konfiguriert werden.

### Datenintegrität

- **Nachrichtensignaturen:**
  - Alle Nachrichten müssen mit **JSON Web Tokens (JWT)** signiert werden.
  - Der JWT muss im **Header** der MQTT-Nachricht übertragen werden.
  - Der JWT enthält:
    - `iss` (Issuer): Identität des Senders (Gerät oder Controller)
    - `sub` (Subject): Thema der Nachricht
    - `exp` (Expiration Time): Ablaufzeit des Tokens (maximal 5 Minuten in der Zukunft)
    - `signature`: Digitale Signatur mit dem privaten Schlüssel des Senders
- **Verifikation:**
  - Der Empfänger überprüft die Signatur mit dem öffentlichen Schlüssel des Senders.
  - Nachrichten mit ungültiger Signatur oder abgelaufenem Token werden verworfen.

### Sicherheitsrichtlinien

- **Schlüsselmanagement:**
  - Private Schlüssel müssen sicher aufbewahrt werden (z. B. in einem Hardware Security Module).
  - Schlüssel dürfen nicht exportiert oder kopiert werden.
- **Protokollierung:**
  - Sicherheitsrelevante Ereignisse müssen protokolliert werden.
  - Protokolle müssen vor unbefugtem Zugriff geschützt sein.

## Fehlermanagement

- **Standardisierte Fehlercodes:**

  | Fehlercode | Beschreibung                 |
  |------------|------------------------------|
  | `E001`     | Kommunikationsfehler         |
  | `E002`     | Hardwarefehler               |
  | `E003`     | Überhitzung                  |
  | `E004`     | Authentifizierungsfehler     |
  | `E005`     | Autorisierungsfehler         |
  | `E006`     | Ungültiges Nachrichtenformat |

- **Fehlerbehandlung durch den Controller:**

  - **Kommunikationsfehler (`E001`):** Versuch einer erneuten Verbindung. Wenn nicht erfolgreich, Benachrichtigung des Benutzers.
  - **Hardwarefehler (`E002`):** Markierung des Geräts als fehlerhaft. Keine weiteren Steuerbefehle senden. Benachrichtigung des Benutzers.
  - **Überhitzung (`E003`):** Gerät ausschalten, falls möglich. Benachrichtigung des Benutzers.
  - **Authentifizierungs-/Autorisierungsfehler (`E004`, `E005`):** Verbindung sofort abbrechen. Sicherheitsvorfall protokollieren.
  - **Ungültiges Nachrichtenformat (`E006`):** Nachricht verwerfen. Warnung protokollieren.

## Datenprotokollierung und Datenschutz

- **Erfasste Daten:**
  - **Statusinformationen:** Gerätestatus, Stromverbrauch, Leistungsbedarf.
  - **Zeitstempel:** Zeitpunkt der Meldungen.
  - **Fehlermeldungen:** Fehlercodes und -nachrichten.

- **Datenspeicherung:**
  - Daten werden in einer **sicheren Datenbank** gespeichert.
  - Zugriff auf die Datenbank ist nur autorisiertem Personal gestattet.
  - Daten werden für einen Zeitraum von **maximal 2 Jahren** gespeichert.

- **Datenschutzgesetze:**
  - Einhaltung der **Datenschutz-Grundverordnung (DSGVO)**.
  - Benutzer haben das Recht, Auskunft über die gespeicherten Daten zu erhalten.
  - Auf Anfrage müssen Daten gelöscht werden (Recht auf Vergessenwerden).

- **Anonymisierung:**
  - Personenbezogene Daten werden vermieden.
  - Wo notwendig, werden Daten pseudonymisiert.

- **Nutzungszwecke:**
  - Daten werden ausschließlich zur **Energieoptimierung** und **Systemverbesserung** verwendet.
  - Keine Weitergabe an Dritte ohne ausdrückliche Zustimmung des Benutzers.

## Mehrere Controller und Redundanz

### Controller-Identifikation

- Jeder Controller erhält eine eindeutige `controller_id` als UUID v4.

### Controller-Kommunikation

- **MQTT-Topics für Controller:**
  - **Status:** `{prefix}/controllers/{controller_id}/status`
  - **Koordination:** `{prefix}/controllers/coordination`

- **Nachrichtenstruktur für Controller-Status:**

  ```json
  {
    "controller_id": "123e4567-e89b-12d3-a456-426614174000",
    "status": "active",
    "timestamp": "2023-10-01T12:34:56Z"
  }
  ```

### Koordinationsmechanismus

- **Leader Election:**
  - Verwendung des **Raft-Konsensalgorithmus** zur Wahl eines führenden Controllers (Leader).
  - Alle Controller implementieren den Raft-Algorithmus gemäß der Spezifikation in [Diego Ongaro und John Ousterhout, "In Search of an Understandable Consensus Algorithm (Extended Version)"](https://raft.github.io/raft.pdf).

- **Zustandssynchronisation:**
  - Der Leader synchronisiert den Zustand der Geräte mit den Followern.
  - Bei Ausfall des Leaders initiieren die Follower eine neue Wahl.

### Synchronisation

- **Heartbeat zwischen Controllern:**
  - Controller senden alle **5 Sekunden** einen Heartbeat an `{prefix}/controllers/coordination`.
  - **Nachrichtenstruktur:**

    ```json
    {
      "controller_id": "123e4567-e89b-12d3-a456-426614174000",
      "term": 3,
      "role": "leader",
      "timestamp": "2023-10-01T12:34:56Z"
    }
    ```

- **Zustandsinformationen:**
  - Geräteinformationen
  - Aktuelle Steuerbefehle
  - Fehlerzustände

## Nutzung von Retained Messages

- **Registrierungsmeldungen** und **Statusmeldungen** sollten als **Retained Messages** gesendet werden.
- Dies stellt sicher, dass neue Controller oder Geräte den aktuellen Systemzustand erhalten.

## Zusammenfassung

Das **Home Energy Optimization Protocol (HEOP)** bietet eine detaillierte und präzise Spezifikation zur effizienten Steuerung und Optimierung des Energieverbrauchs in Smart Homes. Durch die Verwendung standardisierter Protokolle und Sicherheitsmechanismen kann HEOP in verschiedenen Programmiersprachen implementiert werden, solange die Spezifikation eingehalten wird. Die genaue Definition von Nachrichtenstrukturen, Sicherheitsanforderungen und Kommunikationsabläufen gewährleistet Interoperabilität und Sicherheit im gesamten System.
