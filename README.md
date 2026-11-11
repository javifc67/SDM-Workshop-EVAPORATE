# Trabajo Final — Scientific Workshop (Gestión de Sistemas de Datos Masivos · ETSISI-UPM)

Análisis científico, póster y guion de defensa del artículo **EVAPORATE** (*Language Models Enable Simple Systems for Generating Structured Views of Heterogeneous Data Lakes*, PVLDB 2023).

## 📁 Archivos del Repositorio

- 📊 **[Poster_Cientifico_EVAPORATE.pdf](./Poster_Cientifico_EVAPORATE.pdf)**: Póster científico en formato panorámico (1 página) con los 6 apartados del *Scientific Workshop*.
- 🛠️ **[Poster_Cientifico_EVAPORATE.html](./Poster_Cientifico_EVAPORATE.html)**: Código fuente HTML/CSS editable del póster.
- 📘 **[Memoria_y_Guion_Defensa_EVAPORATE.pdf](./Memoria_y_Guion_Defensa_EVAPORATE.pdf)**: Memoria técnica detallada, ficha de validación para Moodle, guion de exposición oral para 3–4 integrantes y batería de preguntas/respuestas de defensa (Q&A).
- ❓ **[Respuestas_Preguntas_Workshop_EVAPORATE.pdf](./Respuestas_Preguntas_Workshop_EVAPORATE.pdf)**: Documento directo de preguntas y respuestas de los 6 bloques de la guía del Workshop.
- 📄 **[Arora_2023_EVAPORATE_VLDB.pdf](./Arora_2023_EVAPORATE_VLDB.pdf)**: Artículo original publicado en *Proceedings of the VLDB Endowment (PVLDB)*, Vol. 17, No. 2 (2023).
- 📑 **[Propuestas_Papers_Workshop.pdf](./Propuestas_Papers_Workshop.pdf)**: Dossier previo con las 9 propuestas de artículos científicos evaluadas para los 3 bloques de la asignatura.

---

## 📝 Respuestas a las Preguntas de la Guía del Workshop (`4_Workshop.pdf`)

### 1. Identificación

#### 1.1. Título del artículo, autores, año y publicación
- **Título:** *Language Models Enable Simple Systems for Generating Structured Views of Heterogeneous Data Lakes* (Sistema **EVAPORATE**).
- **Autores:** Simran Arora, Brandon Yang\*, Sabri Eyuboglu\*, Avanika Narayan, Andrew Hojel (*Stanford University*), Immanuel Trummer (*Cornell University*) y Christopher Ré (*Stanford University*).
- **Año:** 2023 (publicado en octubre/noviembre de 2023 y presentado en la conferencia internacional *VLDB 2024*).
- **Publicación:** *Proceedings of the VLDB Endowment (PVLDB)*, Volumen 17, Número 2, páginas 92–105 (ISSN: `2150-8097`). Clasificación: **Revista JCR Q1** (*Information Systems*) y **Congreso CORE A\*** (*Very Large Data Bases*).

#### 1.2. Referencia bibliográfica y DOI
- **Referencia bibliográfica (formato IEEE/ACM):**
  > S. Arora, B. Yang, S. Eyuboglu, A. Narayan, A. Hojel, I. Trummer, and C. Ré, "Language Models Enable Simple Systems for Generating Structured Views of Heterogeneous Data Lakes," *Proceedings of the VLDB Endowment*, vol. 17, no. 2, pp. 92–105, 2023, doi: 10.14778/3626292.3626294.
- **DOI oficial:** https://doi.org/10.14778/3626292.3626294
- **PDF Open Access (VLDB):** https://www.vldb.org/pvldb/vol17/p92-arora.pdf
- **Repositorio oficial de código:** https://github.com/HazyResearch/evaporate

---

### 2. Motivación y problema

#### 2.1. ¿Qué problema de investigación aborda?
Aborda el problema de **generar automáticamente vistas relacionales estructuradas (tablas SQL) a partir de Data Lakes heterogéneos** compuestos por documentos semi-estructurados y no estructurados (páginas web en HTML, informes técnicos o médicos en PDF, correos electrónicos en texto plano TXT), **sin contar con datos de entrenamiento etiquetados ni conocimiento previo del esquema**.

Formalmente, dada una colección de $n$ documentos crudos $D = \{d_1, d_2, \dots, d_n\}$, el sistema debe:
1. **Descubrir automáticamente el esquema** de atributos relevantes $A = \{a_1, a_2, \dots, a_m\}$ (*Schema Identification*).
2. **Extraer los $n$ registros estructurados (tuplas)** $R = \{r_1, r_2, \dots, r_n\}$ poblando cada celda de la tabla para cada documento $d_i$ y atributo $a_j$ (*Attribute-Value Extraction*).

#### 2.2. ¿Por qué es relevante?
1. **El 80% de los datos empresariales son no estructurados:** Organizaciones médicas, financieras y científicas acumulan millones de documentos en Data Lakes que son "datos oscuros" (*dark data*): no se pueden consultar con SQL, cruzar con un `JOIN`, ni agregar en un Data Warehouse/OLAP hasta que se estructuran.
2. **El fracaso del Estado del Arte ante el Trilema (Generalidad – Coste – Calidad):**
   - **Sistemas Web/DOM (`DOM-LM`, `Ceres`):** Exigen entrenamiento supervisado en cada dominio y asumen que los documentos son siempre HTML con valores alojados exclusivamente en nodos hoja del árbol DOM. Fracasan por completo en PDFs, texto plano o páginas web complejas (como Wikipedia) donde un atributo combina texto y varios hipervínculos.
   - **Sistemas NLP de Extracción Abierta (`OpenIE6`):** Dependen de analizadores gramaticales (POS tagging, árboles de dependencias) que colapsan en documentos largos o tabulados (como informes de la FDA de 20 páginas) y producen esquemas caóticos sin normalizar (427 relaciones distintas en un solo documento FDA).
   - **Pipelines ETL manuales o LLMs directos:** Escribir parsers manuales requiere cientos de horas de ingeniería por cada fuente; y pasar cada documento del Data Lake por un LLM comercial tiene un coste computacional y económico lineal $O(n)$ inasumible (procesar $10.000$ documentos cuesta $\$5.157$ de media y procesar los $55$ millones de artículos de Wikipedia costaría $\$1,1$ millones).

---

### 3. Propuesta

#### 3.1. ¿Cuál es la principal contribución?
1. **Descubrimiento y caracterización de un nuevo compromiso (trade-off) Coste–Calidad con LLMs:** Demuestran que en vez de usar el LLM para procesar directamente todos los documentos del Data Lake (`Evaporate-Direct`, coste lineal $O(n)$), es mucho más eficiente y preciso usar el LLM para **sintetizar código extractor en Python** sobre una muestra diminuta de documentos y ejecutar ese código en CPU sobre todo el Data Lake (`Evaporate-Code+`, coste sublineal $O(m \cdot k)$ constante respecto a $n$).
2. **El sistema `Evaporate-Code+`:** Un sistema simple basado en solo **6 plantillas de prompt agnósticas al dominio** que sintetiza múltiples funciones Python diversas por atributo y las combina estadísticamente, logrando simultáneamente **reducir el coste en tokens $110\times$** y **superar en calidad (+10,1 puntos F1) al propio LLM directo**.
3. **El Algoritmo 1 de Supervisión Débil (Weak Supervision) para código generado por IA:** Extiende la teoría clásica de *Weak Supervision* (pensada para funciones diseñadas por humanos con precisión $>50\%$ y clases discretas) para que funcione sobre funciones generadas por LLMs que devuelven texto libre y donde el **51% de las funciones candidatas tiene calidad $<50\text{ F1}$**, acompañándolo de una garantía teórica de complejidad muestral (**Proposición 1**).

#### 3.2. ¿Qué método, arquitectura o técnica utilizan?
`Evaporate-Code+` opera en un pipeline de **3 etapas** (más una etapa previa de clustering si hay múltiples fuentes mezcladas):
- **Etapa 0 (Opcional, para Data Lakes multi-fuente sin etiquetar):** Vectoriza los documentos con **TF-IDF** y aplica **clustering K-Means jerárquico** para particionar automáticamente el Data Lake en grupos de documentos con estructura homogénea.
- **Etapa 1 — Síntesis del Esquema (*Schema Synthesis*):** Toma una muestra aleatoria $\tilde{D} \subset D$ de solo $k = 10$ documentos, los divide en *chunks* y usa el `Prompt 1` en el LLM para identificar atributos candidatos explícitos. Luego los normaliza y re-rankea por frecuencia para priorizar los atributos de alta cobertura (*head attributes*).
- **Etapa 2 — Síntesis de Código Diverso (*Diverse Code Synthesis*):** Para cada atributo $a_i \in A$, selecciona pasajes de la muestra donde aparece la palabra clave del atributo y pide al LLM que escriba múltiples funciones Python `def get_attribute(doc: str) -> str` combinando dos plantillas (`P_A` orientada a expresiones regulares `re` y `P_B` con ejemplos que importan librerías como `BeautifulSoup`).
- **Etapa 3 — Agregación de Funciones con Supervisión Débil (*Function Aggregation - Algoritmo 1*):** Ejecuta todas las funciones candidatas en CPU ($0,00025\text{ s}$/doc) y aplica el **Algoritmo 1** sobre una muestra de validación $D_{\text{eval}}$ ($\le 10$ docs) comparando cada función con la extracción directa del LLM $\mathcal{F}$:
  1. **Manejo de abstenciones (tasa $e$):** Estima la frecuencia $e$ del atributo en $D_{\text{eval}}$ para distinguir cuándo una cadena vacía es una abstención ($e > \tau$) o una predicción legítima de valor nulo ($e \le \tau$).
  2. **Filtrado de funciones ruidosas:** Descarta todas las funciones con calidad empírica $\hat{a}_j \le 0.5$.
  3. **Discretización (*Top-$b$ Bucketing*):** Agrupa las cadenas de texto votadas por documento en los *top-$b$ buckets* más frecuentes.
  4. **Modelo Gráfico Latente:** Aprende sin supervisión los pesos de confianza y correlaciones de las funciones filtradas $F'$ para emitir el valor consensuado $\hat{y}_i$.

---

### 4. Evaluación y resultados

#### 4.1. ¿Cómo validan su propuesta?
Validan la propuesta en dos regímenes: **ClosedIE** (esquema de atributos dado) y **OpenIE End-to-End** (descubrimiento automático del esquema + extracción de valores), comparando `Evaporate-Code+` frente a `Evaporate-Direct`, `Evaporate-Code`, sistemas NLP abiertos (`OpenIE6`) y modelos supervisados del estado del arte en web (`DOM-LM`, `ZeroShotCeres`, `RoBERTa-Structural`), además de estudios de ablación de cada paso del Algoritmo 1.

#### 4.2. ¿Con qué datos, experimentos o métricas?
- **Datos:** 16 benchmarks reales de 5 dominios y 3 formatos: *SWDE Movies* (8 webs HTML), *SWDE Universities* (5 webs HTML), *Wikipedia NBA Players* (100 páginas HTML complejas), *Enron Emails* ($500.000$ correos TXT) y *FDA 510(k) Medical Device Reviews* (100 informes médicos PDF convertidos a TXT de ~20 páginas).
- **Modelos:** 4 LLMs (`text-davinci-003`, `GPT-4`, `Anthropic Claude-v1` y `Jurassic-2 Jumbo Instruct`).
- **Métricas:** **Text F1** (F1 a nivel de tokens para ClosedIE), **Pair F1** (F1 sobre tripletas `(documento, atributo, valor)` para OpenIE), número de tokens procesados, coste en dólares ($) y tiempo de ejecución en CPU.

#### 4.3. ¿Cuáles son los principales resultados?
1. **Reducción de coste de $110\times$ (y $378\times$ en tamaño real):** Por cada $10.000$ documentos, reduce el consumo medio de $258\text{M}$ de tokens ($\$5.157$) en `Evaporate-Direct` a solo $1,9\text{M}$ de tokens ($\$39$) en `Evaporate-Code+`, con un tiempo de ejecución en CPU de $0,25\text{ ms}$ por función y documento.
2. **Mejora de calidad sobre el propio LLM directo (+10,1 puntos Pair F1 / +18%):** Alcanza **66.7 Pair F1** medio frente a **56.6** de `Evaporate-Direct` y **42.8** de `Evaporate-Code` (destacando +23,4 puntos en Wikipedia NBA y +17,3 puntos en informes FDA), y alcanza **82.3 Text F1** en ClosedIE.
3. **Superación del Estado del Arte Supervisado sin entrenamiento:** En SWDE supera a `DOM-LM` y `ZeroShotCeres` en **+6% en OpenIE** y **+10% en ClosedIE** sin usar ningún dato etiquetado del dominio.

---

### 5. Análisis crítico

#### 5.1. ¿Qué ventajas y limitaciones presenta?
- **Ventajas:**
  1. **Consistencia determinista y eliminación de alucinaciones:** En los informes FDA, el LLM directo olvida el 27,5% de los atributos por documento, alucina 9,7 atributos inexistentes y renombra el mismo atributo de 4 formas distintas, mientras que el código Python sintetizado mantiene un 78,5% de Precisión y consistencia total entre filas.
  2. **Generalidad multi-formato y auditabilidad:** Funciona en HTML, PDF y TXT con cualquier LLM generador de código, y produce funciones Python legibles y auditables por humanos.
- **Limitaciones:**
  1. **Baja cobertura en textos libres sin patrones sintácticos (Caso Enron):** En *Enron Emails*, `Evaporate-Direct` (`93.8 F1`) supera a `Evaporate-Code+` (`86.9 F1`) porque las expresiones regulares no capturan inferencia semántica libre.
  2. **Puntos de corte en esquemas ultra-anchos ($m > 2.500$) o pocos documentos ($n < 40$):** Como el coste de `Code+` escala con el nº de atributos $m$, deja de ahorrar frente a `Direct` si $n < 40$ o $m > 2.500$.
  3. **Dependencia del LLM como "oráculo" en $D_{\text{eval}}$:** Si el LLM falla sistemáticamente en un atributo complejo dentro de los 10 documentos de validación, el filtro descarta todas las funciones candidatas de esa columna.

#### 5.2. ¿Qué preguntas quedan abiertas?
1. **Arquitectura Híbrida Adaptativa:** Cómo combinar `Evaporate-Code+` en CPU como vía rápida por defecto e invocar selectivamente a `Evaporate-Direct` solo en las celdas donde las funciones Python se abstengan.
2. **Detección automática de *Schema / Layout Drift*:** Cómo detectar en producción que una web ha cambiado su estructura DOM para re-sintetizar solo las funciones afectadas.
3. **Seguridad y *Sandboxing*:** Cómo aislar de forma segura la ejecución de código Python y expresiones regulares generadas por LLMs para evitar bucles infinitos (*ReDoS*) o vulnerabilidades.

---

### 6. Relación con la asignatura

#### 6.1. ¿Qué conceptos de la asignatura aparecen en el trabajo?
- **Bloque I (Almacenamiento de Datos):** Conversión de datos no estructurados en datos accionables (`B1_1_4_Fuentres De Datos.pdf`, diap. 6 y caso FDA en diap. 14), **Web Scraping** con `BeautifulSoup`, `XPath` y `regex` (diaps. 20–32), pipelines **ETL/ELT en Data Lakes** (`B2_2_2_Gestión De Informacion.pdf`, diaps. 9–17) y transformaciones **Model-driven vs. Data-driven** (diaps. 26–27).
- **Bloque II (Recuperación de Información y RAG):** **Chunking** de documentos largos, búsqueda léxica, clustering **TF-IDF + K-Means** (`2_1_1_Indexing_and_Information_Retrieval.pdf` y `2_2_Semantic_Retrieval_RAG.pdf`) y evaluación con **Precision, Recall y F1**.
- **Bloque III (Gobernanza e Interoperabilidad):** Descubrimiento automático de esquemas (*Schema Discovery*) y normalización de atributos (`Block 3.1.pdf` y `[3.2] Block 3.2.pdf`).

#### 6.2. ¿En qué amplía o mejora lo estudiado en clase?
1. **Automatiza la programación manual de Web Scraping:** Sustituye la escritura manual de parsers en `BeautifulSoup` y `regex` por síntesis automática de código mediante LLMs.
2. **Resuelve la limitación de los sistemas ETL de la diapositiva 14 (`B2_2_2_Gestión De Informacion.pdf`):** Donde en clase vimos que *"una función rara vez cubre el 100% de los casos potenciales"*, el paper demuestra cómo combinar estadísticamente múltiples funciones imperfectas con **Supervisión Débil** para pasar de `42.8` a `66.7 Pair F1`.
3. **Une los paradigmas *Data-driven* y *Model-driven*:** Usa un modelo *Data-driven* (el LLM) en tiempo de compilación para sintetizar transformaciones *Model-driven* (código Python) ejecutables a escala en CPU.
