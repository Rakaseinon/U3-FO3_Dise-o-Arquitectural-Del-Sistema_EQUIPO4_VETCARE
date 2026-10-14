# ADR 001: Selección del Patrón de Lógica de Negocio y Arquitectura de Dominio para VetCare 24

* **Estatus:** Aprobado
* **Fecha:** 2026-10-13
* **Autores:** Equipo 4 (Componentes de Lógica de Negocio)

## Contexto y Declaración del Problema
VetCare 24 requiere modernizar su plataforma operativa hacia un sistema digitalizado en la nube. El dominio posee reglas operativas críticas como:
1. Inmutabilidad estricta de expedientes clínicos mediante notas de enmienda auditadas (RN-01).
2. Triage y prioridad de emergencias críticas (RN-02).
3. Validaciones de seguridad para evitar prescripción de alérgenos (RN-04).
4. Liberación de jaulas mediante firmas cruzadas de alta clínica y precuenta auditada (RN-06).

Se requiere definir el patrón de diseño para la capa de lógica de negocio y la organización de los servicios de aplicación.

## Opciones Consideradas
1. **Transaction Script:** Agrupar la lógica en funciones de servicio procedimentales.
2. **Table Module:** Estructurar la lógica con base en las tablas de la base de datos relacional.
3. **Domain Model (Rich Domain Model / DDD):** Modelar entidades y agregados con comportamiento y estado bien encapsulado.

## Decisión Seleccionada
Opción elegida: **Domain Model con Domain-Driven Design (DDD)**.

### Consecuencias Positivas:
* **Encapsulamiento de Reglas de Negocio:** Las validaciones de inmutabilidad y de alérgenos quedan protegidas dentro de las entidades del Dominio (`Paciente`, `Consulta`, `Hospitalizacion`), impidiendo estados inconsistentes en la BD.
* **Testabilidad:** Se facilita el desarrollo de pruebas unitarias sobre los Agregados de dominio sin necesidad de infraestructura de base de datos.
* **Trazabilidad con el Negocio:** Alinear el software con el Lenguaje Ubicuo derivado del levantamiento de requerimientos.

### Consecuencias Negativas / Riesgos:
* Mapeo objeto-relacional (ORM) más complejo.
* Curva de aprendizaje técnica mayor para la coordinación mediante la capa de servicios de aplicación (`Service Layer`).

## Cumplimiento
Esta decisión se reflejará en el repositorio de GitHub mediante la implementación de Agregados en la capa de Dominio y Servicios de Aplicación del proyecto.
