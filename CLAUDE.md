# CLAUDE.md

Interaktive 3D-Spiralgehäuse (Schnecke, Muschel, Nautilus) mit Three.js. Eine einzige statische Seite ohne Build-Schritt, veröffentlicht über GitHub Pages: https://helmutqualtinger.github.io/schneckenhaus/

## Struktur

- `index.html`: die gesamte Anwendung (CSS, UI, JavaScript-Modul). Three.js 0.170.0 kommt per Import Map vom jsDelivr-CDN.
- `manifest.webmanifest` + `sw.js`: Progressive Web App (Installation auf iOS/Android, Offline-Cache).
- `assets/`: Screenshots für die README, `social-preview.png` (1280×640, für og:image/twitter:image), `icons/`.

## Aufbau von index.html

- `PRESETS`: Grundformen Schnecke, Nautilus, Muschel, Ammonit (Geometrieparameter, Standardmuster, Innenmaterial). `crossRibs`/`crossRibAmp` erzeugen Querrippen über die Windung (Ammonit).
- `PATTERNS` + `PATTERN_DEFAULTS`: die 13 Muster. Fehlende Felder werden aus den Defaults ergänzt. Flaggen (`flag: 'ps' | 'ru'`, Definitionen in `FLAGS`) füllen die sichtbare Außenhälfte der Röhre (Kachelzeilen 0,25–0,75, Oberkante zur Spitze), eine Flagge pro Kachel ≈ eine Windung.
- `makeShellTextures(pattern, size)`: berechnet Farb- und Reliefkarte pixelweise auf einem Canvas. Alle Terme müssen in u und v periodisch sein (ganzzahlige Frequenzen, periodisches Noise), sonst entstehen Nähte. Ausnahme ist der zelluläre Automat, der nur in v periodisch ist.
- `buildShellGeometry(params)`: Raup-Modell. Spitze fest bei θ = 0, das Gehäuse wächst zur Mündung. Hinter der Spitze werden versteckte Windungen erzeugt (`theta0`), dann schließt Zeile 0 im Grenzpunkt (scharfe Spitze).
- `rebuild(refit)` / `fitView(animated)`: Beim Ziehen des Längenreglers bleibt der Maßstab fest. Erst beim Loslassen (`change`-Event) wird weich neu eingepasst.
- Export: `buildSolid(opts)` baut den massiven Körper einmal für alle Formate (höhere Auflösung über `params.res`, Außenfläche um die Reliefkarte verschoben, Innenfläche um die Wandstärke versetzt, beides auf 0,2·a bzw. 0,45·a des Röhrenradius begrenzt, Rand an der Mündung; gespiegelte Muschelklappe → Windung umdrehen). Vertices über „Grid-Index“ adressiert; `mergeOf` verschmilzt Nahtspalte und Spitze für 3MF/OBJ (getrennte Texturkoordinaten), GLB nutzt das ungemergte Grid. Exporter: `export3MF` (Materials-Erweiterung, `texture2dgroup`), `exportOBJ` (ZIP via fflate), `exportGLB` (GLTFExporter), `exportSTL` (Materialise-Farben). Texturkoordinate v wie in three.js (v = 0 unten, `imageSampler` liest `1 - v`). Prüfen mit trimesh: wasserdicht, konsistente Windung, positives Volumen.
- Pattern-Picker: Vorschaubilder werden mit `makeShellTextures(..., 128)` erzeugt.

## Konventionen

- UI-Texte auf Deutsch.
- Keine Abhängigkeiten außer Three.js vom CDN; kein Build-Tooling.
- Bei neuen Three.js-Addons die URL auch in `sw.js` (`CORE`) eintragen und `CACHE` hochzählen.
- Nach Änderungen an `index.html` die Cache-Version in `sw.js` erhöhen, wenn Offline-Nutzer die neue Version sofort bekommen sollen. Die Seite selbst wird network-first geladen.

## Lokal testen

```bash
npx live-server --port=5500   # nicht Port 5000 (macOS AirPlay)
```

URL-Parameter für reproduzierbare Ansichten: `?preset=nautilus&pattern=zelt&psi=7&turns=8&T=3&clean` (`clean` blendet das Bedienfeld aus).

## Screenshots neu erzeugen

Mit Playwright (Python) und dem installierten Chrome (`channel="chrome"`), Service Worker blockiert. Mit `?clean`-URLs rendern und in `assets/` ablegen. Für `social-preview.png` einen 1280×640-Viewport verwenden: Ammonit (`?preset=ammonit&clean`), Titel als HTML, „3D“ als extrudierter Schriftzug (`TextGeometry`, helvetiker_bold) in die Szene über `window.shellApp` eingefügt. Für Bilder im Desktop-Layout mindestens 641 px Breite verwenden (darunter greift das Handy-Layout).
