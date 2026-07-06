# Guia d'estils — Entrenament dels herois

Inspiració visual: superherois d'animació moderna (anime). Blau fosc i groc de l'armadura, tons neutres freds per a la lectura, targetes que "floten" i línies de velocitat de fons.

Aquesta guia és la font de veritat de l'estil. Els valors es materialitzen com a variables CSS a [index.html](index.html) (bloc `:root`).

## 🎨 Paleta de colors

| Rol | Nom | Hex | Variable CSS | Ús |
|---|---|---|---|---|
| Principal | Blau heroi | `#1A365D` | `--blau` | Navbar, peu de pàgina, títols principals |
| Secundari | Groc energia | `#FACC15` | `--groc` | Botons principals (CTA), icones i subratllats destacats |
| Accent | Blau cel | `#38BDF8` | `--cel` | Estats *hover*, enllaços, decoracions dinàmiques |
| Fons | Gris clar / blanc trencat | `#F8FAFC` | `--fons` | Fons general de l'app |
| Text | Blau nit | `#0F172A` | `--tinta` | Tot el text |

Colors funcionals que es mantenen (codifiquen les tres dimensions, no s'han de tocar):

- Mental → blau `#0C447C` sobre `#E6F1FB`
- Físic → coure `#712B13` sobre `#FAECE7`
- Espiritual → verd `#085041` sobre `#E1F5EE`

## 🔤 Tipografia

Fonts de Google Fonts, carregades per CDN.

- **Títols (`h1`–`h3`)**: `Poppins`, sans-serif geomètrica amb pes. Bold (700) o extra-bold (800) per a l'impacte tipus còmic.
- **Text (body)**: `Nunito`, sans-serif de cantons arrodonits, amigable i molt llegible en pantalla.
- Fallback: `system-ui, -apple-system, sans-serif` si les fonts no carreguen.

## 📐 Elements de la interfície

**Botons principals (CTA — classe `.gran`)**
- Fons groc `#FACC15`, text blau heroi `#1A365D` en negreta.
- Cantonades arrodonides: `border-radius: 8px`.
- *Hover*: petit salt cap amunt (`translateY(-2px)`) i lluïssor; transició suau.

**Botons secundaris (`.secundari`, `.petit`)**
- Fons blanc, vora fina; en *hover*, la vora passa a blau cel `#38BDF8`.

**Targetes (`.targeta`)**
- Fons blanc pur `#FFFFFF`.
- Ombra suau difuminada perquè "flotin": `box-shadow: 0 4px 6px rgba(0,0,0,0.1)`.
- Vora hairline molt subtil i cantonades de 14px.

**Navbar (navegació inferior)**
- Fons blau heroi `#1A365D`, enllaços en to clar; l'element actiu, en groc energia.

**Línies de velocitat**
- Línies diagonals primes de blau cel / gris molt clar al fons d'algunes seccions, per imitar l'efecte de vent i velocitat de les imatges. Sempre subtils, mai per sota de text dens.

## Elements gràfics (carpeta `assets/`)

Els personatges i el logotip, en WebP optimitzat (originals PNG a `assets/originals/`, fora de git):

- `escut.webp` / `escut-icona.webp` — l'escut amb l'estrella daurada. És la **marca**: icona d'app (favicon, PWA, `apple-touch-icon` en PNG a `escut-icona-{180,192,512}.png`) i logotip petit al costat del títol.
- `aina.webp` — la heroïna. `jan.webp` — l'heroi. Imatges d'acció horitzontals, s'usen com a **bàner** (pantalla d'accés) i com a **il·lustració** a la celebració en marcar una sessió com a feta i als estats buits. S'alternen perquè apareguin tots dos.

Regla d'ús: poques imatges i amb aire al voltant. Els personatges no s'usen com a icones petites (són massa detallats); l'escut sí.

## Accessibilitat

- El groc només porta text blau heroi a sobre (mai text clar): és l'única combinació amb prou contrast.
- El blau cel s'usa per a *hover* i decoració, no per a text petit sobre blanc (contrast insuficient); els enllaços de text són blau heroi i canvien a blau cel en *hover*.
