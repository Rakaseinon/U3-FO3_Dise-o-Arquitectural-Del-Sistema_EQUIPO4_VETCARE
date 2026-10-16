# ADR-001: Selección del Patrón de Lógica de Negocio y Arquitectura de Dominio para VetCare 24

* **Fecha:** 2026-10-13
* **Autores:** Equipo 4 — Componentes de Lógica de Negocio (Giovani Patiño, Aldo Romero, Leonardo Muñiz)
* **Proyecto:** Sistema de Gestión Clínica y Hospitalaria 24/7 (VetCare 24)

---

## 1. Contexto y Declaración del Problema

El sistema **VetCare 24** está diseñado para gestionar operativamente una clínica y centro hospitalario veterinario con atención médica ininterrumpida (24/7). El modelo de negocio abarca desde atención primaria de urgencias y agenda diferenciada, hasta el control de expedientes clínicos inmutables, la administración de biológicos y la hospitalización crítica con monitoreo continuo.

Durante el análisis del modelo operativo (AS-IS) y el levantamiento de requerimientos con el Director Médico y el Jefe de Operaciones, se identificaron las siguientes **Reglas de Negocio Inexpugnables (RN-01 a RN-06)**:

1. **RN-01 (Inmutabilidad Médica Estricta):** Una nota de evolución clínica (`NotaSOAP`) firmada por el veterinario se congela de manera inalterable. No se admiten ediciones directas ni borrados; cualquier corrección debe instanciar una `NotaEnmienda` (adenda) vinculada con registro explícito de cédula, fecha, hora y motivo.
2. **RN-02 (Prioridad de Triage Rojo / Código Rojo):** Los pacientes en riesgo vital inminente saltan la fila de espera y los trámites administrativos de cobro para ingresar de inmediato a estabilización en quirófano o sala de choque, generando un pre-expediente temporal de intervención.
3. **RN-03 (Trazabilidad Obligatoria de Biológicos):** Es imposible asentar la aplicación de una vacuna sin capturar formalmente el código de `LoteBiologico` y la `MarcaVacuna` para fines de farmacovigilancia.
4. **RN-04 (Bloqueo Severo por Alergias Activas):** El sistema debe interrumpir automáticamente la confirmación de una receta médica si el principio activo prescrito coincide con un alérgeno registrado en el perfil del paciente, impidiendo accidentes de mala praxis o choques anafilácticos.
5. **RN-05 (Propiedad del Historial Clínico):** El expediente le pertenece a la mascota. Ante una transferencia de titularidad (cambio de `TutorResponsable`), el historial de diagnósticos, cirugías y alergias se conserva intacto.
6. **RN-06 (Validación Cruzada de Alta Hospitalaria):** La liberación física de una jaula hospitalaria exige la convergencia de dos firmas independientes: el dictamen de `AltaMedicaFirmada` por el médico tratante y la confirmación de `CuentaLiquidada` emitida por recepción tras auditar la precuenta de consumos.

**El problema arquitectónico a resolver es:** ¿Qué patrón de diseño debe gobernar la capa de lógica de negocio para garantizar el cumplimiento transaccional de estas reglas, evitar fugas de lógica de seguridad clínica hacia la base de datos o la interfaz de usuario, y mantener el sistema escalable y altamente testable a largo plazo?

---

## 2. Impulsores Arquitectónicos (Architectural Drivers)

El diseño del software está fuertemente restringido por atributos de calidad y requerimientos funcionales críticos:

* **Seguridad y Auditoría Clínica (Safety & Auditability):** Imposibilidad de alterar notas pasadas o recetar fármacos contraindicados.
* **Consistencia Transaccional (Transactional Consistency):** La validación cruzada de egreso hospitalario (RN-06) y la trazabilidad de lotes (RN-03) deben ejecutarse atómicamente.
* **Rendimiento en Urgencias (RNF-01):** Apertura y despliegue del expediente de triage en menos de 3 segundos.
* **Desacoplamiento Tecnológico:** La lógica médica no debe depender de bibliotecas de interfaz gráfica ni del ORM/motor de base de datos relacional.
* **Mantenibilidad y Testabilidad Unitaria:** Capacidad de probar las reglas de vacunación, dosis y alergias de forma aislada mediante pruebas unitarias puras.

---

## 3. Opciones Consideradas

Se evaluaron tres patrones fundamentales de arquitectura para la capa de lógica de negocio (según el catálogo de Martin Fowler):

### Opción 1: Transaction Script (Patrón Procedimental)
Organizar la lógica en funciones o procedimientos lineales por cada transacción del sistema (e.g., `PrescribirMedicamentoScript()`, `AltaHospitalariaScript()`).

* **Pros:** Simplicidad inicial y baja curva de aprendizaje.
* **Contras:** La lógica de validación de alergias (RN-04) e inmutabilidad (RN-01) se duplica en múltiples scripts. Con el tiempo degenera en código "espagueti", haciendo frágil la aplicación y propensa a saltarse reglas de seguridad médica.

### Opción 2: Table Module (Orientado a Tablas)
Estructurar la lógica de negocio alrededor de las tablas relacionales de la base de datos (e.g., una clase de negocio por cada tabla de la BD como `PacientesTableModule`, `ConsultasTableModule`).

* **Pros:** Encaja naturalmente con bases de datos relacionales y estructuras `DataSet`/tabulares.
* **Contras:** Fuerte acoplamiento al esquema físico de la base de datos relacional. Dificultad para modelar entidades complejas compuestas y reglas entrelazadas en tiempo real como la validación cruzada en hospitalización (RN-06).

### Opción 3: Domain Model (Rich Domain Model / Domain-Driven Design - DDD)
Modelar un red de objetos de dominio donde las entidades y agregados encapsulan tanto el estado como el comportamiento y las invariantes del negocio, guiados por un **Lenguaje Ubicuo**.

* **Pros:** Las reglas de negocio (RN-01 a RN-06) residen dentro de los propios Agregados (`PacienteAggregate`, `ConsultaAggregate`, `HospitalizacionAggregate`, `ProfilaxisAggregate`), autoprotegiéndose contra estados inválidos. Altísima testabilidad pura sin base de datos.
* **Contras:** Mayor complejidad conceptual inicial, necesidad de patrones de mapeo de persistencia (Repositories) y curva de aprendizaje para el equipo.

---

## 4. Matriz Comparativa de Evaluación

| Criterio de Evaluación | Transaction Script | Table Module | Domain Model (DDD - Rich Domain) |
| :--- | :--- | :--- | :--- |
| **Complejidad Tecnológica** | Baja | Media | Alta |
| **Encapsulamiento de Invariantes (RN-01 a RN-06)** | Vulnerable (Riesgo de eludir validaciones) | Moderado (Acoplado a tablas relacionales) | **Robusto e Inexpugnable** (Protegido en la Raíz del Agregado) |
| **Mantenibilidad a Largo Plazo** | Baja (Degrada a código espagueti) | Media | **Alta** (Desacoplado de infraestructura) |
| **Testabilidad Unitaria** | Dificultad para aislar la base de datos | Requiere BD relacional para probar | **Excelente** (Pruebas unitarias sobre objetos puros) |
| **Representación del Lenguaje Ubicuo** | Nula (Estructurado por consultas) | Parcial (Estructurado por tablas) | **Total** (Mapeo directo del dominio médico) |
| **Garantía de Inmutabilidad (RN-01)** | Difícil de auditar | Riego de UPDATE directo en tabla | **Garantizada** (Sin setters públicos, adendas explícitas) |

---

## 5. Decisión Seleccionada

Se selecciona la **Opción 3: Domain Model bajo la metodología Domain-Driven Design (DDD)**.

### Componentes de la Solución Adoptada:

1. **Descomposición en 4 Agregados Delimitados (Aggregates):**
   * **`PacienteAggregate`:** Protege la identidad única, el peso actualizado, el microchip y la colección inalterable de alérgenos.
   * **`ConsultaAggregate`:** Garantiza la inmutabilidad de la `NotaSOAP`, la emisión de `NotaEnmienda` auditada y la prescripción farmacológica.
   * **`HospitalizacionAggregate`:** Modela el ciclo de vida de la estancia 24/7, la bitácora de signos vitales cada 2-4 horas y la confirmación cruzada de egreso.
   * **`ProfilaxisAggregate`:** Asegura la captura obligatoria de lote/marca (RN-03) y calcula los refuerzos futuros.

2. **Capa de Servicios de Aplicación delgada (`Service Layer`):**
   * Coordina los flujos de uso (`US01` a `US15`), gestiona transacciones mediante `UnitOfWork` y delega la ejecución de reglas de negocio al Dominio sin ejecutar validaciones directas.

3. **Domain Services y Eventos de Dominio:**
   * Uso de `ServicioValidacionAlergias` y `ServicioAltaHospitalaria` para coordinar flujos complejos inter-agregado y publicación de eventos como `BloqueoAlergiaDisparadoEvent` y `AltaMedicaAutorizadaEvent`.

---

## 6. Consecuencias del Diseño

### Consecuencias Positivas:
* **Autoprotección del Dominio:** Imposible persistir una receta con sustancias alérgicas o dar de alta a un paciente sin firma médica y saldo en cero, ya que los agregados arrojan excepciones de dominio antes de tocar la base de datos.
* **Auditoría e Inmutabilidad Médica:** Cumplimiento normativo de la norma legal médica al forzar el uso de notas de enmienda vinculadas.
* **Pruebas Rápidas y Confiables:** Cobertura de pruebas unitarias sobre las reglas clínicas en milisegundos, al no depender de frameworks o infraestructura de persistencia.
* **Alineación con Stakeholders:** Trazabilidad directa entre el código fuente, los diagramas UML y las definiciones vertidas por el cuerpo médico durante las entrevistas.

### Consecuencias Negativas y Estrategias de Mitigación:
* **Riesgo 1: Complejidad en el Mapeo Objeto-Relacional (ORM).**
  * *Mitigación:* Se implementa el patrón **Repository** aislando las clases de mapeo físico de las entidades del dominio rico.
* **Riesgo 2: Fugas de Lógica de Negocio hacia los Servicios de Aplicación.**
  * *Mitigación:* Revisiones de código estrictas para asegurar que la `Service Layer` solo orqueste (cargar agregado $\rightarrow$ invocar método del agregado $\rightarrow$ guardar agregado).
* **Riesgo 3: Curva de aprendizaje técnica para nuevos desarrolladores.**
  * *Mitigación:* Documentación estandarizada del Lenguaje Ubicuo y diagramas de clases y secuencia integrados en la solución.

---

## 7. Lineamientos de Gobernanza y Cumplimiento

Esta decisión arquitectónica se valida y monitorea bajo los siguientes criterios de ingeniería:

1. **Pruebas Unitarias del Dominio:** Mantenimiento de una cobertura mínima del 90% en métodos de agregados que evalúen reglas de negocio (`TieneAlergiaA()`, `EsAptoParaEgreso()`, `Firmar()`).
2. **Revisión de Código (Code Reviews):** Rechazo automático de PRs que incluyan lógica condicional de negocio dentro de controladores API o clases de la `Service Layer`.
3. **Estructura de Archivos en Repositorio GitHub:** Separación limpia de capas bajo arquitectura limpia (Domain, Application, Infrastructure, UI).

---

