# PROCESS.md: Registre del Procés de Desenvolupament "FineAI" amb IA

Aquest document detalla de manera estructurada el procés de desenvolupament, planificació i col·laboració tècnica amb Intel·ligència Artificial sota la metodologia de Specification-Driven Development (SDD) per a la creació de la PWA de finances personals "FineAI".

---

## 1. Exemples de Prompts utilitzats en la Planificació i Disseny

La col·laboració s'ha basat a utilitzar la IA com a col·laboradora tècnica i no com a simple generadora, demanant explicacions de disseny i especificacions per a eines de prototipatge com **Stitch**.

### Prompt 1: Especificació de la UI a Google Stitch
```text
Prototipar una interfície web mòbil premium de disseny fosc glassmorphic per a una aplicació de finances personals anomenada FineAI.
El fons del canvas és negre obsidiana profund (#0B0F19).
Els components tenen un fons translúcid de color blau marí fosc de rgba(20, 27, 45, 0.6) amb un marcat efecte de desenfocament (blur de 12px) i vores primes blanques translúcides de 1px.
La capçalera superior conté un saldo net en tipografia gran de color blanc, flanquejat per dues petites caixes per a Ingressos (text verd cian #10B981) i Despeses (text vermell corall #EF4444).
A sota s'estén un grid dinàmic: una columna esquerra amb un formulari d'entrada minimalista amb inputs transparents de vores arrodonides, i una columna dreta amb una llista d'activitats de despeses de disseny molt elegant.
El disseny ha de ser totalment fluid, net, professional, sense utilitzar cap icona o emoji no estàndard, centrant-se exclusivament en la qualitat visual dels colors de neó sobre el fons fosc.
```

### Prompt 2: Creació de l'Especificació del Sistema (SPEC.md)
```text
Actua com a Arquitecte de Software Senior i ajuda'm a redactar un document SPEC.md en català que serveixi com a font de veritat absoluta per a una PWA de finances personals anomenada FineAI en Nuxt 3. Ha d'incloure el comportament funcional, actors, user journeys, esquema JSON de dades per a LocalStorage i directrius estrictes de disseny glassmorphic premium obtingudes a partir del prompt de Stitch. No utilitzis cap emoticona o emoji per garantir un perfil estrictament professional.
```

---

## 2. Descripció de les Iteracions amb l'Agent

El procés de construcció s'ha dividit en dues grans iteracions coordinades pel director tècnic (l'alumne):

* **Iteració 1: Definició i Contracte (Fase 1)**: Es va definir el fitxer `SPEC.md` detallant les estructures de dades del pressupost i de les transaccions de forma organitzada, assegurant la coherència en LocalStorage.
* **Iteració 2: Desenvolupament Frontend i Integració Visual (Fase 2)**: Es van implementar els components Nuxt 3 (`app.vue` i `TransactionList.vue`). Es va integrar el disseny procedent de Stitch utilitzant variables de CSS natiu per als tons obsidiana, verd, vermell i cian, dotant l'aplicació d'una identitat visual de molt alta gamma, amb disseny totalment adaptatiu per a mòbils.

---

## 3. Exemple de Bug Solucionat: Hydration Mismatch i ReferenceError en SSR de Nuxt 3

### El Problema Detectat
Durant les primeres proves d'execució de la PWA en entorn local, l'aplicació fallava de manera crítica a la consola llançant errors de tipus:
`ReferenceError: localStorage is not defined` i avisos insistents de `[Nuxt] Hydration node mismatch`.

### L'Explicació Tècnica
Nuxt 3 utilitza de manera nativa el renderitzat pel costat del servidor (SSR - Server Side Rendering). Això significa que el codi del setup dels components s'executa primer al servidor (Node.js) i després es "hidrata" al client (navegador).
1. En el servidor, l'objecte global `window` i la seva API de magatzem de dades `localStorage` **no existeixen**.
2. Intentar carregar directament les dades de transaccions a l'àmbit superior del setup mitjançant `localStorage.getItem('fineai_transactions')` feia caure el procés del servidor de forma fatal.
3. A més, com que el servidor renderitzava un estat inicial de 0 transaccions i el navegador del client local en disposava de dades desades prèviament, es produïa un desajust de nodes HTML de renderitzat (Hydration Mismatch).

### La Solució Aplicada
Per solucionar-ho de manera professional, es van aplicar dues accions correctores:
1. **Encapsulació en el hook `onMounted()`**: Es va moure tota la càrrega inicial de dades des de LocalStorage cap a l'interior d'aquest hook, el qual només s'executa una vegada el component ja s'ha muntat al client, evitant la seva execució al servidor.
2. **Comprovació d'entorn `process.client`**: En el watcher que desa les transaccions, es va afegir una guarda condicional `if (process.client)` per assegurar que només s'intenta escriure a LocalStorage des del costat del navegador.

#### Fragment de codi corregit a `app.vue`
```vue
<script setup>
import { ref, onMounted, watch } from 'vue'

const transactions = ref([])

// Corregit: Només s'executa al client una vegada muntat el component
onMounted(() => {
  if (process.client) {
    const savedTransactions = localStorage.getItem('fineai_transactions')
    if (savedTransactions) {
      transactions.value = JSON.parse(savedTransactions)
    }
  }
})

// Corregit: El watcher només intenta escriure si estem al navegador
watch(transactions, (newVal) => {
  if (process.client) {
    localStorage.setItem('fineai_transactions', JSON.stringify(newVal))
  }
}, { deep: true })
</script>
```

---

## 4. Comparativa Visual: Disseny Generat (Stitch) vs. Resultat Final

* **Disseny Proposat en Stitch**: Una proposta en format fosc amb caixes glassmorphic transparents grises i vores primes de 1px amb un desenfocament profund (`blur`), combinat amb colors de neó verd, vermell i cian per destacar dades clau de despeses i botons de control interactius.
* **Resultat Final Integrat**: S'ha traslladat fidelment el concepte a través de codi CSS natiu de gran precisió. S'ha configurat una variable CSS global amb gradients radials per al fons obsidiana profund, vores amb efecte de vidre fent servir `backdrop-filter: blur(12px)` i transicions molt suaus per als botons de neó i les barres de pressupost interactives. La barra de progrés del límit de pressupost canvia a un degradat d'alerta vermell/taronja quan se supera el 90% d'ús del pressupost, aportant un valor visual afegit de primer nivell.

---

## 5. Reflexió Final sobre el Procés AI-Driven

* **Avantatges de la Metodologia**: El desenvolupament dirigit per especificacions (SDD) recolzat per IA redueix el temps de llançament a producció de manera dràstica. Permet enfocar-se en el disseny, el flux de negoci i l'arquitectura global mentre la IA s'encarrega d'escriure codi de components bàsics.
* **Limitacions i Necessitat de Control Tècnic**: La IA, actuant com a programadora júnior, té mancances greus en conèixer els detalls del cicle de vida de renderitzat asíncron de frameworks com Nuxt 3 (com s'ha demostrat amb el bug de `hydration` o la càrrega inicial). Sense el criteri humà de Tech Lead, és fàcil acabar amb un codi inestable que no compilarà o fallarà de manera silenciosa.
* **Conclusió**: La competència clau del desenvolupador modern és coordinar les capacitats de la IA, establir regles i restriccions explícites (com la font de veritat de SPEC.md) i validar críticament cadascun dels lliurables.
