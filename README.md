# U3-FO3_Dise-o-Arquitectural-Del-Sistema_EQUIPO4_VETCARE
Diseño arquitectónico y modelo de dominio (DDD) para VetCare 24 — Sistema de gestión clínica veterinaria 24/7 con arquitectura orientada a la lógica de negocio.
# 🐾 VetCare 24 — Sistema de Gestión Clínica y Hospitalaria

![Estado del Proyecto](https://img.shields.io/badge/Estado-En_Desarrollo-blue)
![Módulo](https://img.shields.io/badge/FIME_UANL-Componentes_de_Lógica_de_Negocio-green)

Plataforma de gestión clínica e intrahospitalaria veterinaria diseñada bajo el enfoque de **Domain-Driven Design (DDD)** y **Rich Domain Model**. El sistema resuelve deficiencias operativas de registros fragmentados, inmutabilidad médica, seguimiento profiláctico y monitoreo $24/7$.

---

## 📋 Tabla de Contenidos
- [Contexto del Proyecto](#-contexto-del-proyecto)
- [Integrantes del Equipo](#-integrantes-del-equipo)
- [Entregables Arquitectónicos](#-entregables-arquitectónicos)
- [Reglas de Negocio Clave](#-reglas-de-negocio-clave)
- [Estructura del Repositorio](#-estructura-del-repositorio)

---

## 🏢 Contexto del Proyecto
**VetCare 24** opera como un centro hospitalario veterinario ininterrumpido ($24/7$). Su modelo abarca desde consultas generales y especialidades hasta hospitalización continua bajo un esquema de cuenta familiar (Tutor–Mascotas).

### Problemática Resuelta (AS-IS $\rightarrow$ TO-BE)
- **Inmutabilidad y Auditoría:** Prevención de alteración de historiales clínicos mediante notas de enmienda selladas.
- **Seguridad Farmacológica:** Bloqueo preventivo de recetas ante alérgenos registrados en el paciente.
- **Atención Urgente (Triaje):** Carril rápido de admisión para "Código Rojo" que omite trabas administrativas.
- **Validación de Alta Hospitalaria:** Salida de jaula restringida hasta contar con firma de alta médica y liquidación de cuenta.

---

## 👥 Integrantes del Equipo
* **Giovani Aldahir Patiño Rodríguez** — *2118917*
* **Leonardo Alonso Muñiz Espinosa** — *2115635*
* **Aldo René Romero Parga** — *2118748*

**Curso:** Componentes de Lógica de Negocio  
**Cátedra:** M.A. Gómez Marroquín — *FIME UANL*

---

## 📐 Entregables Arquitectónicos

El diseño del sistema se divide en 4 bloques principales:

1. **Patrones de Lógica de Negocio:**
   - Selección y comparación del patrón **Domain Model (DDD)** sobre *Transaction Script* y *Table Module*.
2. **Mapa de Dominio (DDD) & Lenguaje Ubicuo:**
   - Identificación de **Agregados** (`Paciente`, `Consulta`, `Hospitalizacion`, `Profilaxis`), Entidades, *Value Objects* y *Domain Events*.
3. **Capa de Servicios (Service Layer):**
   - Métodos de aplicación para la orquestación pura de los casos de uso sin contaminación de infraestructura.
4. **Diagramación UML & ADR:**
   - Diagrama de clases de dominio completado.
   - Diagramas de secuencia para los flujos complejos (RN-04 Bloqueo de Alergias y RN-06 Alta Hospitalaria).
   - `ADR-001` (Architecture Decision Record) formalizando la elección arquitectónica.

---

## 🛡️ Reglas de Negocio Clave (BRs)
- **RN-01 (Inmutabilidad Médica):** Las notas SOAP firmadas son inalterables. Cualquier corrección requiere una *Nota de Enmienda* auditada.
- **RN-02 (Prioridad Triage Rojo):** Las admisiones de emergencia saltan la lista de espera y crean pre-expedientes inmediatos.
- **RN-03 (Trazabilidad de Biológicos):** Captura obligatoria de número de lote y marca en vacunas.
- **RN-04 (Bloqueo por Alergia):** Interrupción automatizada al intentar recetar sustancias registradas como alérgenos.
- **RN-06 (Validación Cruzada de Alta):** Liberación de jaula condicionada a doble firma (médica + liquidación financiera).

---
