# Roadmap — Generador de sessions d'entrenament integral

Procés iteratiu: cada iteració acaba amb una app **funcional i usable**, es valida amb ús real abans de començar la següent, i es tanca amb un tag de git. El brief de disseny és a [generador-sessions-brief.md](generador-sessions-brief.md).

## Metodologia i git

- **`main` sempre funcional**: cada commit a `main` deixa l'app en estat usable.
- **Commits petits i freqüents**, un canvi lògic per commit.
- **Un tag per iteració tancada**: `v0.1`, `v0.2`... El tag només es posa quan la validació de la iteració ha passat.
- **Porta de qualitat abans de cada tag**: la bateria de tests d'invariants del generador (integrada a l'app com a ruta oculta `#/tests`, sense dependències, mostra verd/vermell al navegador) ha de passar sencera, a més de la validació d'ús real. Els tests creixen amb cada iteració.
- **Branques curtes opcionals** per a increments grossos; per a canvis petits es treballa directament a `main` (projecte d'una sola persona).
- **GitHub Pages** publica `main` des de la iteració 1, perquè la validació real es fa al mòbil.
- La validació de cada iteració és **ús real amb els nens**, no només proves tècniques. Si la validació revela problemes, es corregeixen abans de passar a la següent iteració.

> Nota: aquest ordre substitueix conscientment el de la secció 8 del brief. El CRUD d'activitats passa després del generador perquè el valor a validar primer és "pocs clics → sessió feta", i per això n'hi ha prou amb el catàleg d'exemple.

---

## Iteració 0 — Fonaments ✅

Repositori git inicialitzat amb la documentació de disseny (brief + roadmap) i `.gitignore`.

## Iteració 1 — MVP: "obro, clico, tinc sessió" (`v0.1`)

L'app mínima que valida la idea central.

**Abast:**
- `index.html` amb les tres capes (dades / generador / UI) i el catàleg d'exemple (8-10 activitats) carregat al primer arranc.
- Generador: càlcul de blocs, filtres (edat per dates de naixement, lloc, intensitat acumulativa), regla dels 5 minuts amb descart de dimensió (a l'atzar de moment, sense historial encara), aleatorietat real, ajust final de temps, missatge clar si falten activitats.
- Pantalla **Generar** (durada, lloc, intensitat, botó gros) i pantalla **Sessió** (tres blocs, substituir activitat, regenerar).
- Pantalla **Paràmetres** mínima: dates de naixement dels fills.
- Tests d'invariants del generador a la ruta `#/tests` (les tres dimensions cobertes o descart explícit, cap activitat duplicada dins la sessió, minuts dins del rang de cada activitat, filtres d'edat/lloc/intensitat respectats, error clar quan el catàleg no dona per una combinació).
- Publicació a GitHub Pages.

**Fora del MVP:** historial i no-repetició, CRUD d'activitats, import/export, offline.

**Validació:** durant ~1 setmana, fer 2-3 sessions reals amb els nens des del mòbil. Criteris: del gest d'obrir l'app a tenir sessió en menys de 30 segons; sessions coherents (durades quadren, activitats aptes per l'edat); dues generacions seguides donen sessions diferents.

## Iteració 2 — Memòria: historial i no-repetició (`v0.2`)

**Abast:**
- «Marca com a feta» → escriu a l'historial (amb dimensions treballades i minuts per activitat).
- Exclusió d'activitats fetes fa < X dies i penalització entre X i Y dies (configurables a Paràmetres).
- Rotació de la dimensió descartada en sessions curtes segons historial (substitueix l'atzar de la it. 1).
- Prioritat d'obertura/tancament cap a dimensions poc cobertes.
- Pantalla **Historial** (llistat de sessions fetes).

**Validació:** una setmana d'ús; les sessions consecutives no repeteixen activitats recents i cap dimensió queda sistemàticament oblidada.

## Iteració 3 — Catàleg propi: CRUD d'activitats (`v0.3`)

**Abast:**
- Pantalla **Activitats**: llistat amb cerca i filtres, alta, edició i esborrat, amb validació de camps.
- Protecció contra pèrdua de dades (confirmació d'esborrat, escriptura validada a localStorage).

**Validació:** migrar els recursos reals (enllaços guardats) al catàleg — 15-20 activitats pròpies — i generar sessions només amb elles. Aquesta és la prova de foc del model de dades.

## Iteració 4 — Sincronització: import/export JSON (`v0.4`)

**Abast:**
- Export de tot (catàleg + historial + paràmetres) a un fitxer JSON descarregable, amb opció d'exportar sense historial.
- Avís en exportar: el fitxer conté dades personals (dates de naixement, historial d'ús) i cal guardar-lo en lloc segur.
- Import amb fusió per `id` (actualitza existents, afegeix noves, no esborra res) i validació del fitxer abans d'aplicar-lo.
- Còpia automàtica de les dades locals abans d'aplicar qualsevol import (recuperable si l'import surt malament).

**Validació:** cicle complet PC → fitxer → mòbil i tornada, sense pèrdua ni duplicats.

## Iteració 5 — Poliment i offline (`v1.0`)

**Abast:**
- Repàs UX mòbil: mides tàctils, llegibilitat, estats buits, missatges d'error amables.
- Service worker (`sw.js`, única excepció al fitxer únic) per funcionar offline de veritat.
- Revisió final de tot el flux i neteja de codi.

**Validació:** ús quotidià sense friccions; l'app s'obre i genera sessions en mode avió.

---

## Després de la v1.0 (idees, sense compromís)

Les de la secció 9 del brief: sincronització via Airtable, variacions de descripcions amb LLM, estadístiques d'ús, export PDF, multi-perfil, recordatoris. Es prioritzaran segons l'ús real.
