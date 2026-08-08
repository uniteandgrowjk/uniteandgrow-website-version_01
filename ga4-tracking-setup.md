# GA4-Tracking & Consent Mode v2 — Setup-Dokumentation

Referenzdatei für unite & grow GmbH. Dokumentiert die Tracking-Einrichtung für die Übergabe
an Entwickler oder Mitarbeitende. GA4 ist **direkt per gtag.js** eingebunden — der Google
Tag Manager wird **nicht** mehr verwendet (Container `GTM-PMP5WV2B` wurde am 08.08.2026
vollständig aus allen Seiten entfernt).

## Konfigurationsdaten

| Feld | Wert |
|---|---|
| GA4-Measurement-ID | `G-P70RPB0QCD` |
| Einbindung | direkt per `gtag.js` im `<head>` (kein Tag Manager) |
| Consent Mode | v2, Default = `denied`, Update bei Einwilligung |
| IP-Anonymisierung | `anonymize_ip: true` im `config`-Aufruf |
| Domain | https://www.uniteandgrow.de |
| Eingebaut in | `site/index.html`, `site/impressum.html`, `site/datenschutz.html`, `site/agb.html`, `site/404.html` sowie in der Design-Datei `unite-and-grow-variante-2.dc.html` |

## Wo der Code im Projekt liegt

- **Consent Mode v2 Default:** ganz oben im `<head>`, direkt nach `<meta charset>` und VOR gtag.js.
- **GA4 (gtag.js + config):** unmittelbar danach im `<head>`.
- **Cookie-Banner (UI + Update-Logik):** vor dem schließenden `</body>`.

Es handelt sich um statische HTML-Seiten ohne zentrale Layout-Datei — die Blöcke liegen
daher pro Seite identisch vor.

## Code (Referenz)

```html
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('consent', 'default', {
    'analytics_storage': 'denied',
    'ad_storage': 'denied',
    'ad_user_data': 'denied',
    'ad_personalization': 'denied'
  });
</script>
<script async src="https://www.googletagmanager.com/gtag/js?id=G-P70RPB0QCD"></script>
<script>
  gtag('js', new Date());
  gtag('config', 'G-P70RPB0QCD', { 'anonymize_ip': true });
</script>
```

Beim Klick auf „Zustimmen" im Cookie-Banner:

```js
gtag('consent', 'update', { 'analytics_storage': 'granted' });
localStorage.setItem('cookie_consent', 'granted');
```

Beim Seitenaufruf:

```js
if (localStorage.getItem('cookie_consent') === 'granted') {
  gtag('consent', 'update', { 'analytics_storage': 'granted' });
}
```

## Consent Mode v2 — Verhalten

- **Default (vor Einwilligung):** `analytics_storage`, `ad_storage`, `ad_user_data`,
  `ad_personalization` = `denied`. GA4 setzt keine Cookies, sendet nur cookielose Pings.
- **Bei „Zustimmen":** `analytics_storage` → `granted`.
- **Bei „Ablehnen":** bleibt auf `denied`; die Entscheidung wird gespeichert, der Banner
  erscheint nicht erneut.
- Speicherort der Entscheidung: `localStorage`, Schlüssel `cookie_consent`
  (Werte `granted` / `denied`).

## Test-Checkliste

- [ ] Vor Einwilligung: keine `_ga`-Cookies gesetzt (DevTools → Application → Cookies)
- [ ] Nach „Zustimmen": `_ga`-Cookie erscheint, GA4-Echtzeitbericht zeigt den Aufruf
- [ ] Nach „Ablehnen": kein `_ga`-Cookie, kein Eintrag im Echtzeitbericht
- [ ] Jeder Seitenaufruf erzeugt genau EIN `page_view` (kein doppelter `config`-Aufruf)
- [ ] Alle Unterseiten inkl. 404 laden den Block
- [ ] `anonymize_ip` im `config`-Aufruf vorhanden

## Mögliche Fehlerquellen

- **Doppeltes Tracking:** Keine zweite GA4-Einbindung ergänzen (weder GTM-Tag noch ein
  zweiter `gtag('config', …)`-Aufruf) — sonst werden Seitenaufrufe doppelt gezählt.
- **Consent-Reihenfolge:** Der `consent default`-Block muss VOR dem Laden von gtag.js stehen.
- **Falsche ID:** `G-…` ist die Measurement-ID. `GTM-…`-Container werden hier nicht mehr genutzt.

## Nach dem Netlify-Deployment manuell einzurichten (Google Search Console)

1. Property für `https://www.uniteandgrow.de` anlegen (Domain- oder URL-Präfix-Property).
2. Inhaberschaft bestätigen — per **DNS-TXT-Eintrag** (Domain-Property) oder per
   **HTML-Datei-Upload** bzw. **HTML-Tag**. Die frühere Bestätigung über den GTM-Container
   steht nicht mehr zur Verfügung.
3. `https://www.uniteandgrow.de/sitemap.xml` unter „Sitemaps" einreichen.
4. GA4 mit der Search Console verknüpfen (GA4: Verwaltung → Verknüpfungen → Search Console).
5. Domain-Variante (mit/ohne `www`) als bevorzugt festlegen; Weiterleitung auf Netlify prüfen.
