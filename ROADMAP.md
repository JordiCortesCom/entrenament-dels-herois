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

## Iteració 1 — MVP: "obro, clico, tinc sessió" (`v0.1`) — construïda, validació d'ús en curs

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

## Iteració 2 — Memòria: historial i no-repetició (`v0.2`) — construïda, validació d'ús en curs

> Nota d'implementació: si l'exclusió per historial deixa alguna categoria sense candidates, el generador **relaxa l'exclusió** (tria entre les recents) en lloc de fallar; l'error només apareix quan el catàleg no té cap activitat compatible.

**Abast:**
- «Marca com a feta» → escriu a l'historial (amb dimensions treballades i minuts per activitat).
- Exclusió d'activitats fetes fa < X dies i penalització entre X i Y dies (configurables a Paràmetres).
- Rotació de la dimensió descartada en sessions curtes segons historial (substitueix l'atzar de la it. 1).
- Prioritat d'obertura/tancament cap a dimensions poc cobertes.
- Pantalla **Historial** (llistat de sessions fetes).

**Validació:** una setmana d'ús; les sessions consecutives no repeteixen activitats recents i cap dimensió queda sistemàticament oblidada.

## Iteració 3 — Dades compartides: Firebase (`v0.3`) — construïda, validació d'ús en curs

> **Canvi de pla (2026-07-05):** després de construir les v0.1/v0.2 es va decidir que les dades han de ser compartides entre diversos terminals. Les dades passen de `localStorage` a Firebase Firestore (pla gratuït). Restriccions revisades al brief, secció 6. Aquesta iteració va abans que el CRUD perquè el catàleg real ja s'introdueixi directament sobre dades compartides.

**Abast:**
- Projecte Firebase (pla Spark, gratuït): Firestore + Authentication amb un compte familiar (correu + contrasenya).
- Model a Firestore: document de família amb els paràmetres, i subcol·leccions d'activitats i de sessions (mateix esquema JSON versionat que fins ara).
- Regles de seguretat: només el compte familiar autenticat pot llegir i escriure les seves dades.
- Capa de dades de l'app reescrita sobre Firestore; generador i UI no es toquen (per això la capa de dades estava aïllada).
- Pantalla d'accés: credencials un cop per dispositiu, sessió recordada; missatge clar quan no hi ha connexió.
- Migració automàtica: si el dispositiu té dades locals de les v0.1/v0.2, es pugen a Firestore en el primer accés.

**Validació:** les mateixes dades es veuen des de mòbil i ordinador; una sessió marcada com a feta en un terminal apareix a l'historial de l'altre; la migració conserva l'historial i els paràmetres existents.

## Iteració 4 — Catàleg propi: CRUD d'activitats (`v0.4`) — construïda, validació d'ús en curs

**Abast:**
- Pantalla **Activitats**: llistat amb cerca i filtres, alta, edició i esborrat, amb validació de camps.
- Protecció contra pèrdua de dades (confirmació d'esborrat, escriptura validada).

**Validació:** migrar els recursos reals (enllaços guardats) al catàleg — 15-20 activitats pròpies, des de qualsevol dispositiu — i generar sessions només amb elles. Aquesta és la prova de foc del model de dades.

## Iteració 5 — Còpia de seguretat i poliment (`v1.0`)

**Abast:**
- Export de tot (catàleg + historial + paràmetres) a un fitxer JSON descarregable — còpia de seguretat sota control de l'usuari, amb avís de dades personals i opció d'exportar sense historial.
- Import amb fusió per `id` i validació del fitxer, amb còpia automàtica prèvia (recuperació de desastres si el backend o un import fallen).
- Repàs UX mòbil: mides tàctils, llegibilitat, estats buits, missatges d'error amables (inclosos els de connexió).
- Revisió final de tot el flux i neteja de codi.
- *(Descartat el service worker offline: sense connexió no hi ha dades; l'app mostra un missatge honest en lloc de fingir que funciona.)*

**Validació:** ús quotidià sense friccions des de dos terminals; el cicle export → esborrar → import recupera les dades exactes.

---

## Després de la v1.0 (idees, sense compromís)

Les de la secció 9 del brief que segueixen vives: variacions de descripcions amb LLM, estadístiques d'ús, export PDF, multi-perfil, recordatoris. (La sincronització via Airtable queda descartada: la cobreix Firebase des de la v0.3.) Es prioritzaran segons l'ús real.
