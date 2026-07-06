# Generador de sessions d'entrenament integral per a nens

Document de disseny per crear el projecte amb Claude Code.

---

## 1. Context i objectiu

L'usuari té recursos d'activitats i jocs per fer amb els seus fills (majoritàriament enllaços web guardats) i vol una eina que li permeti generar rutines completes d'entrenament integral (mental + físic + espiritual) amb pocs clics.

**Objectiu final:** quan l'usuari vulgui fer una sessió amb els nens, el sistema ha de compondre una rutina sencera de zero basant-se en un catàleg d'activitats definit prèviament.

---

## 2. Requisits funcionals

### Requisits clau
- Les sessions sempre cobreixen les tres dimensions (mental + físic + espiritual)
- La durada de la sessió és variable i es decideix al moment
- Els fills tenen edats diferents i fan la sessió conjuntament (les activitats triades han de ser aptes per al rang d'edat complet)
- El sistema recorda quines activitats s'han fet recentment per evitar repeticions
- Ús indistint en mòbil i ordinador

### Estructura d'una sessió
Cada sessió generada té tres blocs:
1. **Obertura** — activació suau (~10-15% del temps total)
2. **Nucli** — una activitat de cada dimensió: mental, física, espiritual (~70-80% del temps)
3. **Tancament** — calma i integració (~10-15% del temps)

---

## 3. Model de dades

Cada activitat es representa com un objecte JSON:

```json
{
  "id": "resp-flor",
  "nom": "Respiració de la flor",
  "descripcio": "Inspira imaginant que oloraves una flor, expira bufant espelmes",
  "url": "https://exemple.com/respiracio-flor",
  "dimensions": ["espiritual", "mental"],
  "funcio": ["obertura", "tancament"],
  "durada_min": 3,
  "durada_max": 7,
  "lloc": ["interior", "exterior"],
  "edat_min": 3,
  "edat_max": 10,
  "materials": "cap"
}
```

### Valors possibles dels camps

| Camp | Tipus | Valors |
|---|---|---|
| `id` | string | Identificador únic |
| `nom` | string | Nom curt de l'activitat |
| `descripcio` | string | Descripció breu |
| `url` | string | Enllaç a la font original (opcional) |
| `dimensions` | array | Un o més de: `mental`, `fisic`, `espiritual`, `social` |
| `funcio` | array | Un o més de: `obertura`, `nucli`, `tancament` |
| `durada_min` | número | Minuts mínims |
| `durada_max` | número | Minuts màxims |
| `lloc` | array | Un o més de: `interior`, `exterior` |
| `edat_min` | número | Edat mínima recomanada |
| `edat_max` | número | Edat màxima recomanada |
| `materials` | string | Text lliure (`cap`, `paper i llapis`...) |

### Emmagatzematge i versionat

Tot el que persisteix viu sota una única clau de localStorage, `herois.v1`. Aquest objecte arrel és també el format exacte del fitxer d'export:

```json
{
  "versio": 1,
  "parametres": {
    "fills": [{ "nom": "Aina", "naixement": "2018-03-14" }],
    "dies_exclusio": 7,
    "dies_penalitzacio": 21
  },
  "cataleg": [],
  "historial": []
}
```

Cada sessió de l'historial guarda les dimensions realment treballades i els minuts assignats per activitat (necessari per a la no-repetició, la rotació de la dimensió descartada en sessions curtes i les estadístiques futures):

```json
{
  "data": "2026-07-05T18:30:00Z",
  "durada": 30,
  "activitats": [
    { "id": "resp-flor", "bloc": "obertura", "dimensions": ["espiritual"], "minuts": 3 }
  ],
  "dimensions_descartades": []
}
```

Regles de compatibilitat:
- El camp `versio` és obligatori tant a localStorage com als fitxers d'export. En carregar dades d'una versió anterior s'apliquen migracions seqüencials (`migra(dades, deVersio)`), mai es descarten dades.
- Abans d'aplicar un import es fa una còpia automàtica de les dades existents.
- Tota escriptura a localStorage es valida abans de serialitzar (no perdre dades és prioritari).

---

## 4. Interfície d'usuari

### Pantalla 1 — Generar sessió (principal)

Formulari amb:
- Selector de **durada total** (opcions ràpides: 15/30/45/60 min, o entrada lliure)
- Selector de **lloc** (interior / exterior / qualsevol)
- Botó gran **"Genera sessió"**

Les edats dels fills es configuren un cop a Paràmetres i s'apliquen sempre.

### Pantalla 2 — Sessió generada

Mostra la rutina composta amb els tres blocs:
- Bloc d'obertura (nom, durada, descripció, enllaç)
- Bloc de nucli amb tres activitats (una per dimensió, etiquetades visualment)
- Bloc de tancament

Accions per activitat:
- **Substitueix aquesta activitat** (busca una altra compatible)

Accions globals:
- **Regenera sessió sencera**
- **Marca com a feta** (guarda a l'historial i torna a la pantalla 1)

### Pantalla 3 — Gestió d'activitats

CRUD sobre el catàleg:
- Llistat amb cerca i filtres
- Alta / edició / esborrat
- Import/export de totes les dades (catàleg + historial + paràmetres) com a JSON, amb fusió per `id` (backup i sincronització manual entre dispositius)

### Pantalla 4 — Historial

- Llistat de sessions fetes amb data i activitats
- Utilitzat per la lògica de no-repetició

### Pantalla 5 — Paràmetres

- Dates de naixement dels fills (el rang d'edats es calcula automàticament; el nom és opcional i pot ser un àlies)
- Configuració del període de no-repetició (dies)

---

## 5. Lògica de composició

Algorisme al generar una sessió:

1. **Càlcul de temps per bloc**
   - Obertura: ~10% de la durada total (mínim 3 min)
   - Tancament: ~10% de la durada total (mínim 3 min)
   - Nucli: la resta, repartida entre les tres dimensions

2. **Filtres previs** aplicats a tot el catàleg:
   - Compatibilitat d'edat: `edat_min ≤ edat mínima dels fills` **i** `edat_max ≥ edat màxima dels fills`
   - `lloc` compatible amb el seleccionat
   - Excloure activitats fetes fa menys de X dies (per defecte 7)
   - Penalitzar activitats fetes fa entre 7 i 21 dies (menys probabilitat de sortir)

3. **Selecció d'obertura**: entre les activitats compatibles que tenen `obertura` a `funcio`, tria una a l'atzar amb pes segons historial

4. **Selecció del nucli**: per a cada dimensió (mental, física, espiritual):
   - Filtra activitats que tinguin `nucli` a `funcio` i incloguin aquesta dimensió
   - Tria una a l'atzar dins del temps disponible
   - Evita triar la mateixa activitat més d'un cop dins la sessió
   - Si una activitat és multi-dimensió, pot cobrir dues dimensions si no queden alternatives (fallback)

5. **Selecció de tancament**: com l'obertura però amb `funcio: tancament`

6. **Ajust final de temps**: si la suma no coincideix amb la durada demanada, ajusta els minuts de cada activitat dins del seu rang `durada_min`-`durada_max`

7. **Gestió d'errors**: si no hi ha prou activitats en alguna categoria, mostrar un missatge clar suggerint afegir-ne més (no fallar en silenci ni retornar sessions incompletes)

---

## 6. Restriccions tècniques (revisades el 2026-07-05)

- **Un sol fitxer HTML** amb tot el codi propi (HTML + CSS + JS); l'única dependència externa admesa és l'SDK de Firebase carregat per CDN
- **Backend: Firebase Firestore** — les dades són compartides entre tots els dispositius de la família
- **Autenticació: un compte familiar** (correu + contrasenya amb Firebase Auth), introduït un cop per dispositiu i recordat
- Export manual a JSON com a **còpia de seguretat** (deixa de ser el mecanisme de sincronització)
- Disseny responsive: **mobile first**, funcional en escriptori
- Cal connexió a internet per fer servir l'app (l'offline es va descartar conscientment; sense connexió es mostra un missatge clar)
- El `localStorage` queda només com a llegat: les dades existents dels dispositius es migren a Firestore en el primer accés

> **Versió original d'aquestes restriccions (vigent fins a la v0.2):** un sol fitxer sense cap dependència ni servidor, dades al `localStorage`, sense login, funcionament offline. El canvi es va decidir amb l'ús: la prioritat de compartir dades entre terminals va passar per davant de l'autonomia total del fitxer únic.

---

## 7. Dades d'exemple

L'app s'ha de lliurar amb **8-10 activitats d'exemple** al catàleg per poder provar-se immediatament sense necessitat d'introduir dades. Aquestes activitats han de cobrir totes les combinacions de dimensions i funcions perquè el generador funcioni des del primer moment.

---

## 8. Prioritats de desenvolupament

**Ordre suggerit:**
1. Estructura de dades i localStorage
2. Pantalla de gestió d'activitats (per poder crear/veure les dades)
3. Lògica de composició de sessions
4. Pantalla principal + pantalla de sessió generada
5. Historial i lògica de no-repetició
6. Import/export JSON
7. Poliment visual i UX mòbil

---

## 9. Millores futures (fora del MVP)

- Sincronització automàtica via Airtable API (activitats a Airtable, app només llegint)
- Adaptació de descripcions amb un LLM per fer variacions creatives
- Estadístiques d'ús (activitats més fetes, dimensions més treballades, temps total per dimensió al mes...)
- Compartir sessions o exportar-les com a PDF
- Mode multi-perfil per a diferents combinacions de fills
- Notificacions o recordatoris

---

## 10. Decisions aclarides (2026-07-05)

Respostes de l'usuari a les ambigüitats del brief:

1. ~~**Intensitat "normal"**: escala acumulativa...~~ **Superseda per la decisió 10**: el camp `intensitat` s'ha eliminat del tot.
2. **Sessions curtes**: si en repartir el nucli alguna activitat quedaria per sota de **5 minuts**, es treballa amb **una dimensió menys** en aquesta sessió. La dimensió descartada es tria per **rotació segons historial**: es descarta la més treballada recentment, per mantenir l'equilibri al llarg del temps.
3. **Dimensions d'obertura i tancament**: qualsevol activitat amb la funció adient serveix, però si hi ha candidates es **prioritzen les que cobreixen dimensions poc treballades** al nucli (especialment quan s'ha descartat una dimensió per sessió curta).
4. **Edats**: a Paràmetres es guarden les **dates de naixement** dels fills; l'app calcula el rang d'edats automàticament (no cal actualitzar res quan fan anys).
5. **Import/export JSON**: el fitxer inclou **catàleg + historial + paràmetres**. En importar es **fusiona per id**: actualitza activitats existents, afegeix les noves i no esborra res.
6. **Distribució**: l'app es publicarà a una **URL** (p. ex. GitHub Pages) per usar-la al mòbil com a web afegible a la pantalla d'inici. ~~El localStorage continua sent per dispositiu; la sincronització és l'import/export manual.~~ *(Substituït per la decisió 7.)*
7. **Dades compartides (2026-07-05, després de les v0.1/v0.2)**: les dades passen a **Firebase Firestore** amb un **compte familiar** (correu + contrasenya), compartides entre tots els terminals en temps real. L'offline es descarta (l'app requereix connexió). L'export/import JSON es manté només com a còpia de seguretat. Vegeu les restriccions revisades a la secció 6.
8. **Quarta dimensió: social (2026-07-06)**: s'afegeix la dimensió `social` a les tres originals. El nucli de les sessions cobreix totes les dimensions **que tinguin activitats al catàleg**: si una dimensió no té cap activitat de nucli, la sessió es genera igualment sense ella i s'avisa suggerint afegir-ne (degradació en lloc d'error, per no bloquejar els catàlegs existents). La regla dels 5 minuts i la rotació s'apliquen igual amb quatre dimensions: les sessions curtes en descarten més.
9. **Dimensions seleccionables per sessió (2026-07-06)**: a la pantalla de generar, cada dimensió es pot activar o desactivar (commutadors amb el color de la dimensió; mínim una activa). El nucli només cobreix les dimensions triades que tinguin activitats al catàleg. La selecció es guarda a `peticio.dimensions` perquè «regenera» la respecti. Si cap dimensió triada té activitats de nucli, error clar. Per defecte totes actives; `generaSessio` sense el camp `dimensions` continua cobrint-les totes (compatibilitat).
10. **Eliminació de la intensitat (2026-07-06)**: es retira el camp `intensitat` de les activitats i el selector d'intensitat de la pantalla de generar. Ja no es filtra per intensitat. Motiu: simplificar el model; l'edat, el lloc, la durada i les dimensions ja donen prou control. Les activitats existents a Firestore poden conservar un camp `intensitat` residual que s'ignora (s'esborra en editar-les); no cal migració.

---

## 11. Notes finals per al desenvolupament

- **Prioritat absoluta: pocs clics per generar sessió.** El flux "obro app → trio durada → toco generar → tinc sessió" ha de ser ràpid i sense fricció.
- La gestió d'activitats és secundària: s'usa poc, no cal que sigui bonica, però ha de ser prou robusta per no perdre dades.
- **L'aleatorietat ha de ser real**: dues sessions consecutives amb els mateixos paràmetres han de ser diferents.
- L'usuari tindrà nens al voltant mentre fa servir l'app: text llegible, botons grossos, res de menús laberíntics.
