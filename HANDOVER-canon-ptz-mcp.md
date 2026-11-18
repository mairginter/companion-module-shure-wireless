# Übergabe: Canon CR-N100/N300 – automatische Bildregelung mit Claude Code

Stand: 18.11.2026 · Recherche aus einer Claude-Code-Cloud-Session

## Ziel

Farbe, Weißabgleich und Belichtung von Canon CR-N100/CR-N300 im laufenden Betrieb automatisch
nachregeln, damit das Bild auch bei wechselndem Licht gut aussieht. Claude Code soll über einen
MCP-Server darauf zugreifen. Zielplattform: **Windows**.

## Erkenntnisse

### Kamera & Protokoll
- Steuerung über das **Canon XC Protocol**: HTTP-GETs unter `/-wvhttp-01-/` (z. B. `control.cgi`,
  `info.cgi`), Anmeldung per HTTP Digest/Basic, wenn kein Gastzugang erlaubt ist.
  Doku über das Canon Developer Programme: https://en.canon-cna.com/business/imaging-solutions/xc-protocol/
- WB-Modi: AWB, Set A/B, Daylight (5.600 K), Tungsten (3.200 K), Kelvin 2.000–15.000 K, Manuell (R/B-Gain).
- **Wichtig:** Werte, die auf Auto stehen (AWB, Auto-Iris …), lassen sich nicht von außen
  setzen → für die externe Regelung auf Manual stellen.
- Exakte XC-Parameternamen für WB/Belichtung sind **noch nicht verifiziert** (Doku oder echte Kamera nötig).

### Vorhandene Open-Source-Bausteine
- **PTZ Pilot** – https://github.com/josephdadams/ptz-pilot (MIT, TypeScript/Electron, v1.1.0 vom 14.10.2026)
  - Steuergeräte: Gamepads, Xbox per HID, SpaceMouse, DJI-RC-N1, Magicsee R1
  - Protokolle: VISCA (UDP/TCP/seriell), Canon XC (`src/main/canon/xc.ts`), Panasonic AW, Hikvision,
    Hanwha, ONVIF, KXWell – alle hinter einer gemeinsamen `CameraLink`-Schnittstelle
  - WebSocket-API `ws://127.0.0.1:8765` (siehe `docs/api.md`), Companion-Modul vorhanden
  - Kann **nur** Pan/Tilt/Zoom/Fokus/Presets – **kein** WB/Belichtung (laut `docs/ndi-spec.md` auf „später“ verschoben)
  - **Nur macOS.** Laut Autor ist Windows kein Ziel (`docs/ndi-spec.md`).
    Ein Windows-Build kam als PR #3/#4 von außen (bobergsten, 10.11.2026). Beide sind geschlossen und **nicht gemergt**;
    sie ändern nur Build/Release (NSIS, Actions). Der Code hängt kaum an macOS: nur `native/input-monitoring.c`
    und das Dock-Verhalten, beides schon über `process.platform` abgefragt.
- **Companion-Modul `canon-ptz`** (Bitfocus) – Referenz für XC-Befehle.
- **PTZJoystickControl** – https://github.com/AronHetLam/PTZJoystickControl – Windows/macOS, nur VISCA, GPL-3.0.
- Kommerziell: Cyanview, SKAARHOJ (unterstützen WB über R/B-Gain und Kelvin).

## Entscheidungen
- Sprache: **TypeScript** (läuft unter Node.js; MCP-SDK und Companion sind ebenfalls TS).
- PTZ Pilot **nicht** in den eigenen Code übernehmen, sondern daran andocken
  (WebSocket-API) und Bild-Funktionen möglichst als PR beim Original einbringen.

## Geplante Architektur
1. **MCP-Server (TypeScript, Windows-tauglich)** mit Tools wie `get_snapshot`, `get_settings`,
   `set_white_balance`, `set_exposure`, `apply_profile`.
2. **Regelschleife als normaler Code** (nicht Claude): Standbild → Weißpunkt/Histogramm →
   sanft und begrenzt nachregeln (kein „Pumpen“). Claude setzt Ziele und Profile und greift bei Ausnahmen ein.
3. **Joystick/SpaceMouse** später über PTZ Pilot (sobald Windows) oder eine Alternative.

## Offene Punkte / nächste Schritte
- [ ] Autor von PTZ Pilot fragen: Warum wurden die Windows-PRs geschlossen? Würde er einen
      Windows-Build und Bild-Befehle (WB/Belichtung) annehmen?
- [ ] XC-Protokoll-Doku besorgen bzw. Parameter an einer echten CR-N100/N300 ermitteln.
- [ ] Grundgerüst des MCP-Servers anlegen (eigenes Repo).
- [ ] PTZ Pilot (Stand PR #3) lokal unter Windows bauen und mit SpaceMouse + Kamera testen.
