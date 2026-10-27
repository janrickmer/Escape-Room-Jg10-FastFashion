# Escape-Room „FastFashion“ – Hinweise für die Entwicklung

Escape-Room für Politik und Wirtschaft, Jahrgangsstufe 10 (Gymnasium Hessen): Fast Fashion und Globalisierung. Story: Die
Influencerin Kaya prüft einen Werbevertrag der erfundenen Plattform Flitzr, die Schüler:innen sind ihr Fakten-Team. Sechs
Stationen (Bibliotheksraum → Rätselraum, kein Weg zurück), je ein Stempel für das Codewort GLOBUS, am Ende der Arena-Tunnel.
Oberfläche deutsch, Schüler:innen werden geduzt, echte Umlaute und typografische Anführungszeichen („…“).

## Aufbau

- `index.html` – der ganze Raum in einer Datei (CSS, HTML-Gerüst, ein klassisches `<script>`), keine externen Bibliotheken
  oder CDNs. Reihenfolge im Skript: Konfiguration → `STATIONS` (alle Texte und Rätsel als Daten) → Block
  `GLOBUS:BEGIN … GLOBUS:END` → Zustand → Hilfsfunktionen → PDF → Datensatz/Tunnel → Globus/Karte/Stempel → Bildschirme →
  Zwischenstand → `renderStart()`.
- Bildschirme: `start` → (`resume`) → `intro` → `globe` → `libN` → `roomN` → Stempel-Dialog → … → `final`. Keine
  History-Einträge, kein Zurück vom Rätselraum in die Bibliothek.
- Die Rätselräume sind generisch (`renderRoom`/`checkRoom`): Bausteine `baskets`, `mc`, `assign`, `cloze`, `number`,
  `keystone` aus `STATIONS[i].parts`. Inhalte nur dort ändern; Bibliothekstexte, Lösungserläuterungen und „Genaue Daten“
  sind von der Lehrkraft freigegeben und wörtlich übernommen.
- `fonts/` – Anton und Work Sans als woff2 (SIL Open Font License 1.1, Lizenztexte in `fonts/LIZENZEN.txt`).
- `impressum.html` – Impressum & Datenschutz. Anpassen, wenn sich Datenflüsse ändern. Zur Tippzeit-Messung steht hier
  nichts (die steht nur im Datenschutzhinweis des Feibeitsblatts).
- `CNAME` – `escape-room-jg10-fastfashion.janrickmer.de` (GitHub Pages, Branch `main`).

## Ausnahme von der Reihe

Der Raum gehört zur Reihe janrickmer/escape-room-*, hat aber **keinen eingebetteten Feiditor, keine Abgabe bei Moodle und
keine Auswertung im „Check des Feind(t)es“** – also auch kein `REFLECT_TASKS`, `RED_FIELDS`, `ROOM_CONTEXT`, `%ESCAPEDATA`.
Der Endgegner ist das **Feiditor-Feibeitsblatt** (Repository janrickmer/Feibeitsblatt): Der Arena-Tunnel öffnet es im selben
Tab und übergibt den Datensatz im Link. Der Bestätigungscode steht nur in der Überblick-PDF (und im Datensatz).

## Schnittstelle v1 zum Feibeitsblatt

- Datensatz `feibeitsblatt-escape`: Hülle `{kind:"feibeitsblatt-escape", v:1, d, s}`; `d` = JSON-String der Daten,
  `s` = Prüfsumme `fbEscapeHash('FbEscape|v1|' + d)` (64 Bit als 16 Hex, Testvektor `fbEscapeHash('FbEscape|v1|{}')` =
  `0bdda452e8d9ff95`). Die Prüfsumme ist nur ein Indiz gegen versehentliche Änderungen, kein Schutz.
- Felder von `d` in dieser Reihenfolge: `code`, `room` {id, name, url}, `person` {vorname, nachname, geburtsdatum TT.MM.JJJJ,
  klasse}, `confirm`, `word`, `completedAt` (ISO), `stations` [{label, attempts, errors [{item, wrong?, right?}]}],
  `notes` [{label, text = Erläuterung + Leerzeile + „Genaue Daten: …“}], `resumes` [{place, savedAt, loadedAt}].
  Grenzen wie in der CLAUDE.md des Feibeitsblatts (Hülle ≤ 96 KB u. a.).
- Übergabe: Tunnel → `location.assign(FEIBEITSBLATT_URL + '#escape=' + base64url(UTF-8(Hülle)))` (`buildEscapeRecord`,
  `b64url`, `enterTunnel`); Überblick-PDF → eingebettete Datei `feibeitsblatt-escape.json` (`/Type /EmbeddedFile`,
  Kennzeichen `/FbEscape 1`, unkomprimiert, im `/EmbeddedFiles`-Namensbaum und in `/AF`).
- Änderungen an der Schnittstelle nur in beiden Repositorys gemeinsam (neue Version `v`).

## Konstanten

- `FEIBEITSBLATT_URL = 'https://feibeitsblatt.janrickmer.de/'`, `ESCAPE_ROOM = {id:'powi-jg10-fastfashion', name, url}`.
- **`FEIBEITSBLATT_CODE`** – Code des Escape-Blattes im Feibeitsblatt (6 Zeichen ohne Bindestrich). **Eintragen, sobald die
  Lehrkraft das Blatt angelegt hat.** Solange er leer ist, ist der Tunnel gesperrt und die Überblick-PDF trägt `code: ""`.
- Testhaken nur für automatische Tests: Ist vor dem Laden `window.__FF_TEST__ = {feibeitsblattUrl, code}` gesetzt (Playwright
  `addInitScript`), gelten diese Werte. Produktiv nie setzen.
- Labels „Station n · Titel“ (`ROOM_LABELS`, `SOLUTION_NOTES`) nie ändern – die Aufträge an die KI im Feibeitsblatt verweisen
  wörtlich darauf.

## Regeln

- Rätsel: Karten, Optionen und Zuordnungen bei jedem Aufbau neu mischen (`shuffle` mit `crypto.getRandomValues`, nie
  `Math.random`, nie nach Länge sortieren). Freie Felder über `normalizeAnswer`/`wordKey`/`wordMatches`, Zahlenfelder über
  `numberMatches` (Toleranz, Einheiten, Dezimalkomma). Hilfen in drei Stufen (Absatz → Anfang → Lösung), ein Eintrag je
  falschem Element in der angezeigten (gemischten) Reihenfolge. Jeder Versuch zählt; die Fehler des ersten Versuchs stehen
  in Textform (`item`, `wrong`, `right`, in derselben Reihenfolge) in Überblick-PDF und Datensatz.
- Personendaten und Eingaben nur per `textContent` oder `escapeHtml` ins DOM. Kein `alert`, Meldungen stehen im Dokument.
- Kein Server, keine Cookies, kein localStorage, kein `fetch`. YouTube erst nach Klick (Lader im `<head>`, youtube-nocookie).
  Überblick-PDF und Zwischenstand nur als Download (`application/octet-stream`), nie in einem neuen Tab.
- „Copyright Jan-Rickmer Feindt“ auf jedem Bildschirm (Footer) und auf jeder PDF-Seite (Fußzeile).
- Gestaltung: Anton + Work Sans, flache Kontinentfarben (Europa Mint, Südamerika Orange, Asien Himmelblau, Afrika Gelb),
  Globus aus Landpunkten (Natural Earth 1:110m, gemeinfrei, über world-atlas 2.0.2; Fibonacci-Kugel mit 12 000 Punkten,
  Antarktis ausgelassen), keine Verläufe, kein Glas, keine Eyebrows, keine Schatten außer am Stempel-Dialog;
  `prefers-reduced-motion` schaltet alle Bewegung ab. Bildsprache: Kayas Schreibtisch-Globus mit Meridianring und Fuß
  (`globeStand`, viewBox 0 0 400 456), Flugbogen vom Pin der vorigen Station mit Spitze während der Drehung; Stationskopf
  als Band in der Kontinentfarbe (`stationBand`: Papier-Etikett, ab 1200 px der Stempel-Rohling `stampSvg(i, uid, true)`);
  erledigte Kacheln und Reisepass mit Mini-Stempeln, ein neuer Stempel landet im Pass-Dialog und in der Topbar (`landSlot`);
  Routenkarte als Ausschnitt `MAP_VIEW` mit Nummern-Scheiben; Arena-Tunnel mit Rippen, sechs Bahnen in den
  Kontinentfarben und dem Codewort über dem Tor (offen: Bahnen leuchten einmal auf, `prelit`/`lit`). Lesespalte `--read`
  (40rem ≈ 66 Zeichen), Silbentrennung mit `hyphenate-limit-chars`.
- Automatische Tests (Playwright) liegen nicht im Repository. Live für Schüler:innen ist Branch `main`: vor jedem Push den
  ganzen Durchlauf testen und eine echte Überblick-PDF im Feibeitsblatt hochladen.
