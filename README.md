# demo-suiteaudit-qaframework-qaautomationlabs

**Reporte HTML de la auditoría:** https://keber.dev/demo-suiteaudit-qaframework-qaautomationlabs/

Demostración de las capacidades del paquete [`@keber/qa-suite-audit`](https://www.npmjs.com/package/@keber/qa-suite-audit), aplicado con **Claude Code** para auditar la suite de pruebas automatizadas del repositorio [keber/demo-qaframework-qaautomationlabs](https://github.com/keber/demo-qaframework-qaautomationlabs).

## Origen y propósito

`@keber/qa-suite-audit` es una herramienta de auditoría repetible y basada en evidencia para suites de pruebas automatizadas. Implementa la rúbrica de 7 criterios que Adriana Troche Robles publicó en *Calidad sin Humo*: [Cómo saber si tus tests automatizados sirven: la rúbrica de 7 criterios para auditar una suite](https://calidadsinhumo.com/guias/rubrica-para-juzgar-una-suite-de-tests/).

La rúbrica evalúa siete dimensiones en una escala de 0 a 3: Oráculo, Sensibilidad, Determinismo, Diagnóstico, Cobertura de reglas de negocio, Vida en CI y Mantenibilidad. Oráculo y Sensibilidad son eliminatorias.

La herramienta separa a propósito tres roles:

1. **Scripts que producen evidencia** (`evidence`): leen el repositorio y miden, con archivo y línea, sin ejecutar la suite.
2. **Un juez que asigna los niveles** (una persona o un agente), dentro de los límites que la evidencia permite y citándola.
3. **Un validador** (`validate`) que rechaza un veredicto sin citas, fuera de límites o escrito contra otra evidencia.

Un puntaje que no se puede rastrear hasta algo medido es una opinión con un número.

El repositorio auditado es a su vez una demostración: una suite Playwright de 119 casos generada con [`@keber/qa-framework`](https://www.npmjs.com/package/@keber/qa-framework) y Claude sobre la tienda demo [shop.qaautomationlabs.com](https://shop.qaautomationlabs.com/shop.php). El propósito de esta auditoría es mostrar qué tan buena es realmente una suite generada por IA cuando se la mide con una rúbrica externa y con evidencia, más allá de que "todo esté en verde".

## El prompt

Todo el contenido de este repositorio se generó a partir de este prompt:

```text
crea una nueva carpeta: demo-suiteaudit-qaframework-qaautomationlabs, inicializa git en
ella e instala el paquete npm @keber/qa-suite-audit@next , y con él audita el repo
https://github.com/keber/demo-qaframework-qaautomationlabs . En el readme indica que es
una demostración de las capacidades del paquete qa-suite-audit, su origen y propósito;
incluye este prompt inicial, publica los resultados de la auditoría en el readme de ese
repo, y publica el repo en mi github de forma pública con el nombre
demo-suiteaudit-qaframework-qaautomationlabs . Publica también la version html del
reporte en una github page en una rama dedicada, e incluye el link en el readme.
```

> **Nota de versión:** el tag `@next` no existe en npm para este paquete (solo `latest`). Se usó la versión publicada `0.1.0`.

## Cómo se hizo

| Paso | Comando o actor | Resultado |
|---|---|---|
| 1. Fijar el objetivo | `git clone` | Repositorio auditado en el commit `8a2c98e3ca11ef985c03a2538eb89dd54bf775f3` |
| 2. Evidencia | `npx qa-suite-audit evidence ./audited-repo --out audit/input.json` | Adaptador `qa-framework`, 15 archivos de test, digest `6d9a4247...` |
| 3. Juicio | Sub-agente Claude (Opus 5.5) siguiendo `docs/JUDGMENT-PROTOCOL.md` | [`audit/verdict.json`](audit/verdict.json) |
| 4. Validación | `npx qa-suite-audit validate audit/input.json audit/verdict.json` | `VALID   10/16` |
| 5. Reportes | `npx qa-suite-audit report ... --format html` | [`audit/report.html`](audit/report.html), [`audit/report.txt`](audit/report.txt) |

El juez fue un agente sin relación con los que escribieron la suite, con instrucción explícita de ser adversarial y no generoso.

## Resultado de la auditoría: 10 / 16

```text
Suite audit  audited-repo
  adapter qa-framework   evidence 6d9a42476dd3   15 test file(s)
  suite type regression

  RESULT   10/16
           v1 reads the repository only; 16 of 21 points are reachable at all

  D1 Oracle           1/3  #..
  D2 Sensitivity      1/3  #..   calibrated threshold
  D3 Determinism      1/2  #.
  D4 Diagnosis        2/3  ##.
  D5 Rule coverage    2/2  ##
  D6 Life in CI       1/1  #     settled by evidence
  D7 Maintainability  2/2  ##
```

El puntaje se expresa sobre los puntos **alcanzables**, no sobre 21. La versión 1 lee solo el repositorio. Varios niveles de la rúbrica dependen de hechos que un repositorio no puede demostrar:

- la tasa de flakiness entre corridas (D3);
- las exclusiones deliberadas por escrito (D5);
- si el CI bloquea el merge o el despliegue (D6);
- el proceso de revisión y retiro de pruebas (D7).

Por eso solo 16 de los 21 puntos son alcanzables. Además, D2 queda limitado a 2 por calibración al no existir configuración de mutation testing, así que el máximo posible con esta evidencia es 15. Tampoco se aplican las bandas de veredicto de la rúbrica original, que asumen evidencia completa.

Ninguna dimensión eliminatoria quedó en 0, pero **las dos eliminatorias (D1 y D2) quedaron en 1**, un escalón por encima de la eliminación.

| Dimensión | Nivel | Límite | Justificación |
|---|---|---|---|
| D1 Oráculo (eliminatoria) | 1 | 0..3 | Alta trazabilidad a reglas, pero la especificación se derivó de la app en ejecución y varias "reglas" solo congelan el comportamiento observado. |
| D2 Sensibilidad (eliminatoria) | 1 | 0..2 | Sin tests sin asserts, pero hay tests que fallan por causas ajenas a su título (precondiciones sobre datos del catálogo vivo). |
| D3 Determinismo | 1 | 0..2 | Sin esperas fijas, pero `retries: 2` en CI sin justificación absorbe la inestabilidad sin medirla. |
| D4 Diagnóstico | 2 | 0..3 | Trace, video y screenshot en fallo, y títulos de negocio; el reporte nombra el caso, no la regla que se rompió. |
| D5 Cobertura de reglas | 2 | 0..2 | 86 de 105 reglas enlazadas, con 16 casos negativos y 16 de límite; no hay criterio escrito de exclusión. |
| D6 Vida en CI | 1 | 1..1 | Fijado por evidencia: el workflow existe pero es solo `workflow_dispatch` y no bloquea ningún merge. |
| D7 Mantenibilidad | 2 | 0..2 | Nombres de negocio y 90% de locators `data-testid`; capas parciales y duplicación estructural. |

Las rutas siguientes son relativas al repositorio auditado, en el commit indicado.

### Fortalezas principales

- **Pruebas de defecto con polaridad correcta.** Las 6 pruebas de defecto abierto afirman lo que exige la regla, no lo que hace la app. El helper `expectOpenDefect` relanza los errores de infraestructura y falla cuando el defecto deja de reproducirse (`qa/07-automation/e2e/fixtures/defects.ts:149-180`; uso en `tests/cart/checkout.spec.ts:225`).
- **Oráculos concretos en carrito y checkout.** Subtotales y totales con valores exactos (`tests/cart/cart-management.spec.ts:113-119`, `$500 x 3 = $1500`). También verifican propiedades, como el orden ascendente y descendente en `tests/catalog/category-listing.spec.ts:187-199`.
- **Sincronización sin esperas fijas.** Cero `waitForTimeout`, con aserciones web-first y `expect.poll`. Cada test siembra su propio `localStorage`, sin dependencia de orden.
- **Higiene de secretos.** El login solo ocurre en el global setup o en el proyecto `login-flow`, que no graba artefactos (`playwright.config.ts:118` y `:144`).
- **Trazabilidad declarada.** El 93% de los tests lleva ID de caso y el 93% de los casos declara su regla.

### Debilidades principales

1. **El oráculo es en parte una caracterización del sistema.** La especificación se generó inspeccionando la app y varias reglas transcriben lo observado:
   - `RN-CAT-048` (`qa/01-specifications/module-catalog/submodule-cart-badge/01-business-rules.md:55`) declara como comportamiento esperado un error JavaScript no controlado, y `tests/catalog/cart-badge.spec.ts:135` afirma que el error ocurre. Si alguien corrige el bug, el test falla.
   - `tests/catalog/cart-badge.spec.ts:112` congela la falta de sincronización entre pestañas, y `tests/cart/cart-management.spec.ts:181` congela "sin límite máximo de cantidad". Ambas conductas figuran como recomendaciones de usabilidad, no como reglas confirmadas por negocio.
   - `RN-CAT-042` (`01-business-rules.md:16`) dice literalmente "Observado".
2. **Tests que fallan por causas ajenas a lo que protegen.** `openWithBaseCart` (`tests/cart/checkout.spec.ts:41-45`) afirma el total `$800` del catálogo público como precondición de todas las pruebas de validación de formulario (por ejemplo `checkout.spec.ts:66`). Un cambio de precio en el sitio de terceros pone en rojo pruebas sobre mensajes de error. Lo mismo ocurre con los conteos fijos de productos (`openCategory(page, '/mens-wear.php', 20)`).
3. **El CI no bloquea nada.** `.github/workflows/e2e.yml:3-4` solo tiene `workflow_dispatch`: la suite corre cuando alguien la lanza a mano, no en push ni en PR.
4. **Reintentos que ocultan inestabilidad.** `playwright.config.ts:210` define `retries: process.env.CI ? 2 : 0` sin explicación. Un test que pasa al reintentar queda en verde como "flaky" y nada en el repositorio lleva la cuenta. La única cifra de flakiness es un informe de estabilización puntual de 3 corridas.
5. **Pruebas que nunca se ejecutan inflan el conteo.** Los 9 tests de `qa/07-automation/integration` son plantillas del framework que se omiten en tiempo de ejecución (`integration/tests/example.spec.ts:21`, `security/no-session.spec.ts:20`, `write/example-write-cycle.spec.ts:41`). Además, ningún pipeline los invoca.
6. **Duplicación estructural y capas parciales.** Hay solo dos page objects (Login y Checkout). El bloque que recolecta diálogos se repite en `tests/cart/cart-management.spec.ts:39`, `:135`, `:195` y `tests/catalog/cart-badge.spec.ts:88`.
7. **El proyecto `login-flow` no deja artefactos de fallo.** Es un trade-off de seguridad deliberado, pero un fallo de login en CI llega sin trace ni screenshot.

### Recomendaciones priorizadas

1. **Separar regla de observación (D1).** Marcar en la especificación qué reglas confirmó negocio y cuáles son comportamiento observado. Retirar o reescribir las pruebas que afirman bugs como esperados (`cart-badge.spec.ts:126-136`).
2. **Activar el CI como gate (D6).** Agregar disparadores `pull_request` y `push` a la rama principal y exigir la verificación en la protección de rama.
3. **Aislar las precondiciones de datos (D2).** Reportar las verificaciones sobre el catálogo vivo (total `$800`, conteos de productos) como fallos de datos de prueba, no como fallos del comportamiento bajo prueba.
4. **Medir y justificar los reintentos (D3).** Publicar el conteo de tests flaky de cada corrida y hacer fallar el job sobre un umbral, o bajar `retries` a 0.
5. **Medir la sensibilidad (D2).** Romper reglas a propósito o configurar mutation testing y versionar su configuración. Es lo único que la herramienta lee para levantar el tope de D2.
6. **Eliminar o completar la suite de integración** de `qa/07-automation/integration`.
7. **Mostrar la regla en el reporte (D4).** Incluir el ID de la regla (`RN-...`) en el título o en una anotación de cada test.
8. **Reducir duplicación (D7).** Extraer helpers a fixtures y agregar page objects para carrito y catálogo.
9. **Escribir las exclusiones deliberadas (D5).**

### Preguntas para el equipo

La auditoría no responde estas preguntas por el equipo. Solo la primera puede cambiar el puntaje en esta versión, una vez que la respuesta quede versionada en el repositorio y se vuelva a extraer la evidencia.

| ID | Pregunta | Efecto |
|---|---|---|
| D2.sensitivity-measured | ¿Se ejecuta mutation testing contra esta suite en algún lugar, y dónde está su configuración? | Leída por esta versión: permitiría nivel 3 |
| D3.flakiness-measured | ¿Se mide en algún lugar la tasa de fallo de estos tests entre corridas, y dónde se puede consultar? | Solo para el lector |
| D5.deliberate-exclusions | ¿Hay reglas de negocio excluidas de la suite a propósito, y dónde está escrita esa decisión? | Solo para el lector |
| D6.deployment-condition | ¿Una corrida en verde de esta suite es condición para desplegar, y dónde está configurado? | Solo para el lector |
| D6.merge-blocking | ¿Una corrida fallida de esta suite bloquea el merge, y dónde está configurado? | Solo para el lector |
| D7.review-retirement | ¿Existe un proceso para revisar y retirar tests, y dónde está escrito? | Solo para el lector |

## Límites de esta auditoría

- **Solo evidencia del repositorio.** La v1 no ejecuta la suite ni consulta el servidor de CI, la protección de ramas o el historial de corridas.
- **Niveles de juicio.** Un agente asignó los niveles abiertos a partir de una muestra de tests leídos, con confianza media en D1, D2 y D3. Otro juez podría defender un 2 en D1 por la fuerte vinculación a reglas.
- **Umbrales calibrados.** El tope de D2 en 2 proviene de un umbral calibrado (`D2.sensitivity-measured`), no del texto literal de la rúbrica.
- **Conteos que subestiman.** El extractor reporta 0 fallos esperados y 0 tests omitidos, pero hay 6 `test.fail` indirectos (vía `expectOpenDefect`) y 9 tests de integración omitidos en tiempo de ejecución. El juez lo registró como objeción al umbral, sin modificarlo.
- **Citas corregidas tras el juicio.** Una revisión posterior verificó cada referencia archivo:línea del veredicto contra el commit auditado. Cinco citas de `cart-badge.spec.ts` estaban desplazadas 160 líneas (por ejemplo `:295` en vez de `:135`). Se corrigieron sin cambiar ningún nivel y el veredicto se volvió a validar (`VALID 10/16`). El validador comprueba coherencia con la evidencia, no la exactitud de las líneas citadas: esa verificación sigue siendo responsabilidad de quien publica.

## Reproducir la auditoría

Requisitos: Node.js 20 o superior.

```bash
npm install
git clone https://github.com/keber/demo-qaframework-qaautomationlabs.git audited-repo
git -C audited-repo checkout 8a2c98e3ca11ef985c03a2538eb89dd54bf775f3
npx qa-suite-audit evidence ./audited-repo --out audit/input.json
npx qa-suite-audit validate audit/input.json audit/verdict.json
npx qa-suite-audit report audit/input.json audit/verdict.json --format html --out audit/report.html
```

El paso de juicio (`audit/verdict.json`) no lo genera la herramienta: lo escribe una persona o un agente siguiendo `node_modules/@keber/qa-suite-audit/docs/JUDGMENT-PROTOCOL.md`. Si la evidencia extraída cambia, `validate` rechaza el veredicto por obsoleto.

## Estructura

| Ruta | Contenido |
|---|---|
| `audit/input.json` | Evidencia extraída del repositorio auditado y límites por dimensión |
| `audit/verdict.json` | Veredicto del juez, con citas a la evidencia |
| `audit/report.html` | Reporte HTML (publicado en GitHub Pages desde la rama `gh-pages`) |
| `audit/report.txt` | Reporte de terminal |

## Créditos

- Rúbrica: Adriana Troche Robles, [Calidad sin Humo](https://calidadsinhumo.com/). El texto de la rúbrica es obra de su autora.
- Herramienta: [`@keber/qa-suite-audit`](https://www.npmjs.com/package/@keber/qa-suite-audit).
