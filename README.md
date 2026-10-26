# MessageCenter – IP-Symcon-Modul

Das MessageCenter speichert mehrzeilige Textnachrichten bis zur Quittierung und zeigt offene sowie quittierte Einträge in einer eigenen Seite der Webvisu an.

## Funktionen

- Beliebig viele Nachrichten gleichzeitig
- Mehrzeilige Texte werden unverändert mit Zeilenumbrüchen gespeichert
- Erstellungszeitpunkt und Quittierungszeitpunkt je Nachricht
- Quittierte Nachrichten bleiben bis zur Aufbewahrungsfrist erhalten
- Automatische Löschung ausschließlich quittierter Nachrichten
- Die Frist wird ab dem Erstellungszeitpunkt berechnet. Wird eine Nachricht erst nach Ablauf der Frist quittiert, wird sie bei der nächsten Bereinigung entfernt.
- Offene Nachrichten bleiben unabhängig von ihrem Alter erhalten
- Konfigurierbare Aufbewahrung in Tagen (Standard: 30 Tage)

## Skript-API

Die Modulaktion kann aus jedem IP-Symcon-PHP-Skript aufgerufen werden. Ersetze `12345` durch die Instanz-ID des MessageCenter:

```php
IPS_RequestAction(12345, 'AddMessage', "Alarmmonitor: Test\nVariable: Heizung\nBedingung: Wert zu hoch");
```

Für AlarmMonitor-Nachrichten können optionale Zustandsmetadaten als JSON-Umschlag mitgesendet werden. Normale Textaufrufe bleiben unverändert kompatibel:

```php
$payload = json_encode([
    'source' => 'AlarmMonitor',
    'text' => "Meldung: Alarm\nVariable: Heizung",
    'alarmState' => 'initial' // initial, escalation oder resolved
], JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
IPS_RequestAction(12345, 'AddMessage', $payload);
```

Eine Nachricht quittieren:

```php
IPS_RequestAction(12345, 'AcknowledgeMessage', 'NACHRICHTEN_ID');
```

Die Nachrichten-IDs findest du im JSON der Instanzvariablen **Nachrichten**. Jede Nachricht enthält:

- `id`: eindeutige Kennung
- `text`: vollständiger Text einschließlich Zeilenumbrüchen
- `createdAt`: Unix-Zeitstempel der Erstellung
- `acknowledged`: Quittierungsstatus
- `acknowledgedAt`: Unix-Zeitstempel der Quittierung oder `null`
- `alarmState` (optional): `initial`, `escalation` oder `resolved`; wird nur bei AlarmMonitor-Nachrichten gespeichert

## Integration der Webvisu

Die Visu liest die String-Variable **Nachrichten** der Modulinstanz. Für Schreibaktionen nutzt sie `IPS_RequestAction` mit den Aktionen `AcknowledgeMessage` bzw. `AddMessage`. Texte werden in der Nachrichtenliste in einem aufklappbaren Bereich mit `white-space: pre-wrap` dargestellt, sodass mehrzeilige Alarmmonitor-Nachrichten lesbar bleiben. AlarmMonitor-Nachrichten werden anhand von `alarmState` gelb (Erstmeldung), rot (Eskalation) oder grün (Alarm aufgehoben) markiert. Nachrichten ohne dieses Feld behalten die Standarddarstellung.

Die Seite **Nachrichten** wird nur im Seitenmenü aufgeführt. Die Filter Offen/Quittiert bleiben beim Scrollen oben sichtbar. Der Seitenkopf zeigt die Anzahl offener Nachrichten.

## Installation und Konfiguration

1. Das Modul MessageCenter über die Symcon-Bibliothek aktualisieren/laden.
2. Eine MessageCenter-Instanz anlegen.
3. Die Aufbewahrungsfrist quittierter Nachrichten in Tagen einstellen.
4. Die Instanz-ID aus dem Objektbaum in aufrufende Skripte übernehmen.
5. In der Visu die Nachrichten-Seite öffnen; eine eigene Hauptmenüposition ist nicht erforderlich.

## Sicherheit und Verhalten

- Die Modulaktion lehnt leere Nachrichten bzw. unbekannte Aktionen ab.
- Die Bereinigung löscht nur quittierte Nachrichten.
- Die Bereinigung läuft täglich, außerdem nach Änderungen an der Instanz und nach jeder Quittierung.
- Speicherdaten liegen dauerhaft in der String-Variable **Nachrichten** der Instanz.
