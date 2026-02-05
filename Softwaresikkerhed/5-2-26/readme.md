# **Secure Scale Catalog (RBAC) + Scale Classification (Intervals)**

Systemet er et “katalog over skalaer”, hvor en **myndighed** vedligeholder skala-definitioner (CRUD), mens en **elev** kun kan læse (Read/List).

- **Adgangskontrol (RBAC):** Elev må ikke ændre kataloget (forhindrer uautoriserede ændringer / tampering).
- **Inputvalidering (allow-list og fail-closed):** Forhindrer fejltilstande, uforudsigelig parsing og “junk” data.
- **Dataintegritet:** Systemet lagrer kun validerede intervaller og beregnede modes.

---

## **1. Domæne**
### **1.1. Skala-definition**
En skala-definition beskrives med:
- `name` (fx `"Major"`, `"Natural minor"`)
- `intervals` (7 intervaller i halvtoner)

**Regler for `intervals` (diatonisk):**
- Type: `list[int]`
- Længde: 7
- Hvert element: 1 eller 2
- Sum: 12 (en oktav)

**Klassifikation baseret på intervals:**
- `MAJOR` hvis `[2,2,1,2,2,2,1]`
- `MINOR` hvis `[2,1,2,2,1,2,2]`
- Ellers: `UNKNOWN` (hvis intervals er valide men ikke matcher)

### **1.2. Roller og operationer (RBAC)**
Roller:
- `AUTHORITY` (myndighed)
- `LEARNER` (elev)

Operationer på kataloget:
- `CREATE`, `READ`, `UPDATE`, `DELETE`, `LIST`

**RBAC-regel:**
- `AUTHORITY` må alt (CRUDL)
- `LEARNER` må kun `READ` og `LIST`

---

## **2. Ækvivalensklasser**
### **2.1. RBAC (rolle + operation)**
| Klasse | Rolle | Operation | Forventning |
|---|---|---|---|
| EC-R1 | AUTHORITY | CREATE/UPDATE/DELETE | Tilladt |
| EC-R2 | AUTHORITY | READ/LIST | Tilladt |
| EC-R3 | LEARNER | READ/LIST | Tilladt |
| EC-R4 | LEARNER | CREATE/UPDATE/DELETE | Afvist (AccessDenied) |
| EC-R5 | Ukendt rolle | enhver | Afvist (AccessDenied) |

### **2.2. `name` (skala-navn)**
Politik: fail-closed + enkel allow-list/format.
| Klasse | Input | Forventning |
|---|---|---|
| EC-N1 | string, 1..40 tegn, trimmet ikke tom | OK |
| EC-N2 | tom / whitespace | ValueError |
| EC-N3 | ikke string (None/int/list) | ValueError |
| EC-N4 | for langt navn (>40) | ValueError |

### **2.3. `intervals` (primær data)**
| Klasse | Input | Forventning |
|---|---|---|
| EC-I1 | list[int], len=7, værdier {1,2}, sum=12 | OK |
| EC-I2 | ikke liste | ValueError |
| EC-I3 | len<7 | ValueError |
| EC-I4 | len>7 | ValueError |
| EC-I5 | element ikke int | ValueError |
| EC-I6 | element udenfor {1,2} | ValueError |
| EC-I7 | sum != 12 | ValueError |
| EC-I8 | valid men ikke major/minor | OK → mode=UNKNOWN |

---

## **3. Grænseværditest**
### **3.1. `intervals` længde (krav: 7)**
| Test | Input | Forventning |
|---|---|---|
| BV-L1 | len=6 | ValueError |
| BV-L2 | len=7 | OK |
| BV-L3 | len=8 | ValueError |

### **3.2. interval-værdi (krav: 1 eller 2)**
| Test | Input | Forventning |
|---|---|---|
| BV-V1 | 0 | ValueError |
| BV-V2 | 1 | OK |
| BV-V3 | 2 | OK |
| BV-V4 | 3 | ValueError |

### **3.3. sum (krav: 12)**
| Test | Input | Forventning |
|---|---|---|
| BV-S1 | 11 | ValueError |
| BV-S2 | 12 | OK |
| BV-S3 | 13 | ValueError |

### **3.4. name længde (krav: 1..40 efter trim)**
| Test | Input | Forventning |
|---|---|---|
| BV-N1 | "" | ValueError |
| BV-N2 | "A" | OK |
| BV-N3 | 40 tegn | OK |
| BV-N4 | 41 tegn | ValueError |

---

## **4. Decision Table Tests (to tabeller: RBAC + validering/klassifikation)**
### **4.1. Decision Table A – RBAC autorisation**
Betingelser:
- A1: role == AUTHORITY?
- A2: operation in {CREATE, UPDATE, DELETE}?

Handling:
- H1: Tillad
- H2: Afvis (AccessDenied)

| Regel | A1 | A2 | Handling |
|---|---:|---:|---|
| DT-A1 | T | T | H1 |
| DT-A2 | T | F | H1 |
| DT-A3 | F | T | H2 |
| DT-A4 | F | F | H1 |  (LEARNER må READ/LIST)

### **4.2. Decision Table B – inputvalidering + klassifikation**
Betingelser:
- B1: name er str?
- B2: name trimmet længde 1..40?
- B3: intervals er list?
- B4: len(intervals)=7?
- B5: alle elementer int?
- B6: alle elementer ∈ {1,2}?
- B7: sum=12?
- B8: matcher major?
- B9: matcher minor?

Handling:
- K1: ValueError
- K2: mode=MAJOR
- K3: mode=MINOR
- K4: mode=UNKNOWN

| Regel | B1 | B2 | B3 | B4 | B5 | B6 | B7 | B8 | B9 | Handling |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|
| DT-B1 | T | T | T | T | T | T | T | T | F | K2 |
| DT-B2 | T | T | T | T | T | T | T | F | T | K3 |
| DT-B3 | T | T | T | T | T | T | T | F | F | K4 |
| DT-B4 | F | - | - | - | - | - | - | - | - | K1 |
| DT-B5 | T | F | - | - | - | - | - | - | - | K1 |
| DT-B6 | T | T | F | - | - | - | - | - | - | K1 |
| DT-B7 | T | T | T | F | - | - | - | - | - | K1 |
| DT-B8 | T | T | T | T | F | - | - | - | - | K1 |
| DT-B9 | T | T | T | T | T | F | - | - | - | K1 |
| DT-B10 | T | T | T | T | T | T | F | - | - | K1 |

---

## **5. CRUD(L) testcases (praktisk, security-oriented)**

Entity: `ScaleDefinition { id, name, intervals, mode }`

| ID | Operation | Rolle | Input | Forventning |
|---|---|---|---|---|
| CRUD-C1 | CREATE | AUTHORITY | valid name+intervals | Success |
| CRUD-C2 | CREATE | LEARNER | valid name+intervals | AccessDenied |
| CRUD-R1 | READ | AUTHORITY | existing id | Success |
| CRUD-R2 | READ | LEARNER | existing id | Success |
| CRUD-U1 | UPDATE | AUTHORITY | existing id + valid data | Success |
| CRUD-U2 | UPDATE | LEARNER | existing id + valid data | AccessDenied |
| CRUD-D1 | DELETE | AUTHORITY | existing id | Success |
| CRUD-D2 | DELETE | LEARNER | existing id | AccessDenied |
| CRUD-L1 | LIST | AUTHORITY | - | Success |
| CRUD-L2 | LIST | LEARNER | - | Success |

---

## **6. Cycle process test (livscyklus)**
Cycle: Create → Read → Update → Read → Delete → Read

| Step | Operation | Rolle | Forventning |
|---|---|---|---|
| 1 | CREATE | AUTHORITY | record findes |
| 2 | READ | LEARNER | kan læse |
| 3 | UPDATE | AUTHORITY | ændringer gemt |
| 4 | READ | LEARNER | ser opdateret data |
| 5 | DELETE | AUTHORITY | record fjernet |
| 6 | READ | LEARNER | NotFound |

---

## **7. Testpyramiden (hvad testes hvor)**
| Lag | Fokus | Eksempler |
|---|---|---|
| Unit (flest) | inputvalidering + klassifikation | DT-B + grænse/ækvivalens |
| Integration | RBAC + CRUD(L) + cycle | repo + AccessDenied/NotFound |
| E2E (få) | valgfrit (API/CLI) | “learner list/read; authority create/update/delete” |

---