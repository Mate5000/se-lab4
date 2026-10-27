# GT4500 – rendszertesztek

## CLI-próba

| Parancs | Kimenet |
| --- | --- |
| `HELP` | Parancslista és súgó |
| `TORPEDO,SINGLE` | `No ship has been initialized` |
| `GT4500,1,0,1,0` | `SUCCESS` |
| `TORPEDO,SINGLE` | `SUCCESS` |
| `TORPEDO,SINGLE` | `SUCCESS` |
| `TORPEDO,SINGLE` | `FAIL` |
| `VALAMI` | `Unknown command: 'VALAMI'` |
| `TORPEDO,INVALID` | `Unknown firing mode: 'INVALID'` |
| `GT4500,1` | `usage: GT4500,<PRI_CNT>,<PRI_FAIL_RATE>,<SEC_CNT>,<SEC_FAIL_RATE>` |
| `GT4500,1,1,1,0` | `SUCCESS` |
| `TORPEDO,SINGLE` | `FAIL` |
| `TORPEDO,ALL` | `SUCCESS` |
| `EXIT` | Kilépés |

Hibás parancs után a program tovább fut.

## Tesztterv

S = `TORPEDO,SINGLE`, A = `TORPEDO,ALL`.
A sorszám az input-output fájlpár száma.
Az inicializálás külön `SUCCESS` sorát a kimenet oszlopa nem tartalmazza.

| # | Teszt | Inicializálás | Parancsok | Elvárt kimenet |
| --- | --- | --- | --- | --- |
| 1 | Első lövés az elsődlegesből | `GT4500,1,0,1,1` | S | SUCCESS |
| 2 | SINGLE: egy torpedó fogy lövésenként | `GT4500,1,0,1,0` | S, S, S | SUCCESS, SUCCESS, FAIL |
| 3 | Váltás a másodlagosra; hiba után nincs újrapróbálás | `GT4500,2,0,1,1` | S, S | SUCCESS, FAIL |
| 4 | Elsődleges hibája után nincs átváltás | `GT4500,1,1,1,0` | S | FAIL |
| 5 | Üres elsődleges helyett másodlagos | `GT4500,0,0,2,0` | S, S, S | SUCCESS, SUCCESS, FAIL |
| 6 | Üres másodlagos helyett elsődleges | `GT4500,3,0,0,0` | S, S, S, S | SUCCESS, SUCCESS, SUCCESS, FAIL |
| 7 | SINGLE: mindkettő üres | `GT4500,0,0,0,0` | S | FAIL |
| 8 | ALL: mindkét tárból fogy egy torpedó | `GT4500,1,0,1,0` | A, S | SUCCESS, FAIL |
| 9 | ALL: elsődleges hibás, másodlagos működik | `GT4500,1,1,1,0` | A, A | SUCCESS, FAIL |
| 10 | ALL: elsődleges működik, másodlagos hibás | `GT4500,1,0,1,1` | A, A | SUCCESS, FAIL |
| 11 | ALL: mindkettő hibás | `GT4500,1,1,1,1` | A | FAIL |
| 12 | ALL: elsődleges üres, másodlagos működik | `GT4500,0,0,1,0` | A, A | SUCCESS, FAIL |
| 13 | ALL: elsődleges működik, másodlagos üres | `GT4500,1,0,0,0` | A, A | SUCCESS, FAIL |
| 14 | ALL: mindkettő üres | `GT4500,0,0,0,0` | A | FAIL |
| 15 | ALL: elsődleges üres, másodlagos hibás | `GT4500,0,0,1,1` | A | FAIL |
| 16 | ALL: elsődleges hibás, másodlagos üres | `GT4500,1,1,0,0` | A | FAIL |
| 17 | SINGLE: az üres elsődleges helyett próbált másodlagos hibás | `GT4500,0,0,1,1` | S | FAIL |

A 0-s hibaaránnyal a tár mindig működik, 1-essel mindig hibázik, így az
eredmények nem véletlenszerűek. Az üres és a hibás tár külön eset, mert
SINGLE módban csak az üres helyett lehet a másikból lőni.
A több lövésből álló tesztek a váltást és a torpedók elfogyását is ellenőrzik.

## Eredmény

```text
./mvnw verify
Tests run: 21, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

18 rendszerteszt és 3 meglévő egységteszt futott. A teszteken a futtatás után
nem kellett változtatni.

| GT4500 | Eredeti tesztekkel | Bővítés után |
| --- | --- | --- |
| Utasításfedettség | 67,16% | 98,51% |
| Ágfedettség | 33,33% | 95,24% |

A 80%-os követelmény teljesült. Fedetlen maradt a `fireLaser` visszatérése
és a `switch` alapértelmezett ága, amely a két megadható móddal nem érhető el.

[JaCoCo-képernyőkép](kepernyokepek/jacoco-source.png)

## Korlátok

A CLI csak SUCCESS vagy FAIL választ ad. A tárak hívásai és a maradék
torpedószám nem látható közvetlenül. Két működő tár teljes váltogatási
sorrendjét, illetve ALL módban a hibás tár meghívását ezért nem lehet minden
esetben igazolni. Ehhez a tárhívásokat figyelő egységteszt kellene.
