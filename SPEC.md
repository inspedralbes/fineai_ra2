# SPEC.md: Especificació Tècnica i Funcional del Sistema "FineAI"

Aquest document actua com a font de veritat del sistema i defineix de forma estricta l'arquitectura, el disseny, el flux de dades i els components de la PWA de finances personals "FineAI".

---

## 1. Descripció Funcional de l'Aplicació

"FineAI" és una Progressive Web App (PWA) mòbil i d'escriptori dissenyada per ajudar els usuaris a gestionar les seves finances personals de manera intuïtiva. L'aplicació permet registrar transaccions (ingressos i despeses), categoritzar-les, controlar un pressupost límit mensual de forma visual i desar les dades de manera segura en el propi dispositiu.

### Característiques principals
* **Tauler de Control (Dashboard)**: Visualització simplificada del balanç total (ingressos, despeses, saldo net) mitjançant indicadors de gran impacte i una barra de progrés reactiva de pressupost.
* **Registre de Transaccions**: Formulari minimalista per afegir transaccions definint concepte, import, tipus (ingrés o despesa) i categoria (alimentació, transport, habitatge, oci, factures, altres).
* **Gestió Dinàmica de Pressupost**: Control actiu del límit de despeses mensuals amb avisos visuals i canvis cromàtics d'alerta quan el consum supera els llindars de seguretat.
* **Progressive Web App (PWA)**: Aplicació instal·lable al dispositiu de l'usuari, amb capacitat offline completa mitjançant Service Workers i persistència permanent de dades via LocalStorage.

---

## 2. Actors del Sistema

* **Usuari Final**: Persona que instal·la l'aplicació per gestionar les seves finances. Té el control absolut per afegir i eliminar transaccions, definir el seu pressupost límit mensual i rebre els avisos visuals de consum.
* **Navegador Web / Dispositiu Mòbil**: Allotja l'entorn d'execució de la PWA, gestiona la instal·lació mitjançant el Service Worker i emmagatzema l'historial de transaccions a LocalStorage.

---

## 3. User Journey (Flux d'Usuari)

1. **Accés i Benvinguda**: L'usuari obre la PWA al seu navegador. Si és la primera vegada, es mostra un saldo de 0 euros. L'aplicació li proposa instal·lar-se com a PWA en la pantalla d'inici del mòbil o escriptori.
2. **Definició de Pressupost**: L'usuari estableix el seu límit de despeses mensuals (per defecte 1000 euros) de forma instantània prement el botó d'edició ràpida.
3. **Afegir Transaccions**: L'usuari introdueix un ingrés (nòmina) i una despesa (compra mensual), triant la categoria de cada node.
4. **Actualització del Dashboard**: El sistema recalcula en segon pla el balanç net, actualitza els indicadors visuals i modifica la barra de progrés. Si la despesa acumulada s'acosta al límit, la barra canvia de verd a vermell d'alerta.
5. **Eliminar Elements**: L'usuari pot esborrar qualsevol transacció del llistat. El balanç i l'estat del pressupost es tornen a calcular reactivament a l'instant.

---

## 4. Estructura de Dades (Esquema JSON)

Les dades es modelen i s'emmagatzemen a LocalStorage de la següent manera per garantir consistència:

### 4.1. Transaccions (`fineai_transactions`)
```json
[
  {
    "id": "1725519805123",
    "concept": "Nomina mensual",
    "amount": 1800.00,
    "type": "income",
    "category": "nomina",
    "date": "2026-05-28T14:00:00Z"
  },
  {
    "id": "1725519805456",
    "concept": "Compra Supermercat",
    "amount": 75.50,
    "type": "expense",
    "category": "alimentacio",
    "date": "2026-05-28T15:30:00Z"
  }
]
```

### 4.2. Pressupost (`fineai_budget`)
```json
{
  "monthlyLimit": 1000.00
}
```

---

## 5. Especificació per a l'Eina de Disseny Generatiu "Stitch"

Aquest apartat descriu l'especificació visual de disseny requerida per ser introduïda a Google Stitch (o sistemes de generació de disseny de UI equivalents) per exportar les plantilles que donaran vida a l'aplicació.

### 5.1. Codi de Tokens i Paleta de Colors (Tema Fosc Glassmorphic)
* **Fons de l'Aplicació (Canvas background)**: `#0B0F19` (Obsidiana profund amb un gradient radial molt subtil a `#1E2640` al centre).
* **Fons de Targetes i Panells (Card background)**: `rgba(20, 27, 45, 0.6)` amb un efecte de desenfocament de fons (`backdrop-filter: blur(12px)`) i una vora sòlida molt prima de `1px solid rgba(255, 255, 255, 0.08)`.
* **Colors d'Accent de Marca**:
  * Ingressos i Accions Positives: Verd Neó `#10B981` (Emerald).
  * Despeses i Alertes: Vermell Corall `#EF4444`.
  * Elements de Disseny i Enllaços: Cian Elèctric `#06B6D4` (Cyan).
  * Text Principal: `#F3F4F6` (Gris molt clar de gran contrast).
  * Text Secundari: `#9CA3AF` (Gris mitjà).

### 5.2. Disseny de la Interfície (Layout Grid)
L'estructura es divideix en un disseny adaptatiu de dues grans seccions:
1. **Capçalera del Dashboard (Secció superior)**:
   * Targeta de visualització de Saldo Net gran (Tipografia estil Sans-Serif moderna i forta com *Outfit* o *Inter*).
   * Dues sub-targetes més petites sota el Saldo Net: una per a "Ingressos totals" (color d'accent verd) i una altra per a "Despeses totals" (color d'accent vermell).
   * Una barra d'estat de pressupost de disseny degradat que mostra visualment quant queda abans d'arribar al límit mensual de despeses establert per l'usuari.
2. **Cos de l'Aplicació (Secció central de dues columnes a Escriptori, una columna a Mòbil)**:
   * **Columna Esquerra**: Formulari per afegir transaccions (Concept, Import, Tipus en Ràdio Buttons i Categoria en Dropdown) amb inputs minimalistes de fons transparent i vora cian.
   * **Columna Dreta**: Llista de transaccions recents mostrades com a línies fines amb categories identificades amb petites etiquetes de colors i un botó de text d'eliminació de contorns fins.

### 5.3. Prompts proposats per a Stitch
Aquest és el prompt estructurat per introduir a l'eina generativa de disseny:
```text
Prototipar una interfície web mòbil premium de disseny fosc glassmorphic per a una aplicació de finances personals anomenada FineAI.
El fons del canvas és negre obsidiana profund (#0B0F19).
Els components tenen un fons translúcid de color blau marí fosc de rgba(20, 27, 45, 0.6) amb un marcat efecte de desenfocament (blur de 12px) i vores primes blanques translúcides de 1px.
La capçalera superior conté un saldo net en tipografia gran de color blanc, flanquejat per dues petites caixes per a Ingressos (text verd cian #10B981) i Despeses (text vermell corall #EF4444).
A sota s'estén un grid dinàmic: una columna esquerra amb un formulari d'entrada minimalista amb inputs transparents de vores arrodonides, i una columna dreta amb una llista d'activitats de despeses de disseny molt elegant.
El disseny ha de ser totalment fluid, net, professional, sense utilitzar cap icona o emoji no estàndard, centrant-se exclusivament en la qualitat visual dels colors de neó sobre el fons fosc.
```
