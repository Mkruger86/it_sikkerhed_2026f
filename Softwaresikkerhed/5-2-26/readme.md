# Secure Scale Catalog (RBAC) + Scale Classification (Intervals)

Systemet er et “katalog over skalaer”, hvor en **myndighed** vedligeholder skala-definitioner (CRUD), mens en **elev** kun kan læse (Read/List).

- **Adgangskontrol (RBAC):** Elev må ikke ændre kataloget (forhindrer uautoriserede ændringer / tampering).
- **Inputvalidering (allow-list og fail-closed):** Forhindrer fejltilstande, uforudsigelig parsing og “junk” data.
- **Dataintegritet:** Systemet lagrer kun validerede intervaller og beregnede modes.

---

## **1. Domæne: hvad systemet gør**
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

## 2) Ækvivalensklasser (Equivalence Partitioning)

### 2.1 RBAC (rolle + operation)
| Klasse | Rolle | Operation | Forventning |
|---|---|---|---|
| EC-R1 | AUTHORITY | CREATE/UPDATE/DELETE | Tilladt |
| EC-R2 | AUTHORITY | READ/LIST | Tilladt |
| EC-R3 | LEARNER | READ/LIST | Tilladt |
| EC-R4 | LEARNER | CREATE/UPDATE/DELETE | Afvist (AccessDenied) |
| EC-R5 | Ukendt rolle | enhver | Afvist (AccessDenied) |

### 2.2 `name` (skala-navn)
Politik: fail-closed + enkel allow-list/format.
| Klasse | Input | Forventning |
|---|---|---|
| EC-N1 | string, 1..40 tegn, trimmet ikke tom | OK |
| EC-N2 | tom / whitespace | ValueError |
| EC-N3 | ikke string (None/int/list) | ValueError |
| EC-N4 | for langt navn (>40) | ValueError |

### 2.3 `intervals` (primær data)
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
