# 📊 Guía de Métricas Ágiles y Gobernanza de Delivery en Power BI

> **Autor:** Rosmar Mendoza — Ingeniera de Sistemas & Technical Project Manager  
> **Área:** Business Intelligence, Data-Driven Agile Delivery & Gobierno de TI

---

## 1. El Marco de Métricas Ágiles (Más Allá de la Velocidad)

En equipos de alto desempeño, la métrica tradicional de *Velocity* (Story Points) es insuficiente si no se cruza con métricas de flujo y calidad.

```
       MÉTRICAS DE FLUJO                     MÉTRICAS DE CALIDAD                    MÉTRICAS DE NEGOCIO
┌──────────────────────────────┐       ┌──────────────────────────────┐       ┌──────────────────────────────┐
│ • Cycle Time (Días)          │       │ • Escapes a Producción (Bugs)│       │ • Cumplimiento de SLAs (%)   │
│ • Lead Time (Días)           │       │ • Tasa de Retrabajo (%)      │       │ • Time-to-Market de Releases │
│ • Throughput (Items/Semana)  │       │ • Deuda Técnica vs Features │       │ • Certidumbre de Entrega (%) │
│ • WIP (Work In Progress)     │       │ • MTTR (Resolución Fallas)   │       │ • Disponibilidad Plataforma  │
└──────────────────────────────┘       └──────────────────────────────┘       └──────────────────────────────┘
```

---

## 2. Definición y Fórmulas Clave

### A. Cycle Time vs. Lead Time
* **Lead Time:** Tiempo total transcurrido desde que una idea o requerimiento se crea en el backlog hasta que está en producción listo para el usuario.
  $$\text{Lead Time} = \text{Fecha Despliegue Producción} - \text{Fecha Creación Ticket}$$
* **Cycle Time:** Tiempo real de ingeniería desde que un desarrollador mueve el ticket a *In Progress* hasta que pasa la *Definition of Done (DoD)*.
  $$\text{Cycle Time} = \text{Fecha Done} - \text{Fecha Inicio Desarrollo}$$
* *Meta de Gestión:* Reducir la dispersión (varianza) del Cycle Time para lograr estimaciones con un **85% de confianza**.

### B. Diagrama de Flujo Acumulado (Cumulative Flow Diagram - CFD)
Herramienta visual para auditar cuellos de botella:
* **Líneas paralelas:** Flujo saludable y entrega continua constante.
* **Banda que se ensancha:** Acumulación excesiva de Trabajo en Progreso (WIP) en una fase (típicamente *Code Review* o *QA Verification*).
* **Acción del PM:** Aplicar **WIP Limits** estrictos en columnas de revisión para destrabar el flujo antes de ingresar nuevas tareas.

---

## 3. Arquitectura del Dashboard Ejecutivo en Power BI

Diseñado para sincronizar datos de **Jira Software API** y **Datadog** en reportes ejecutivos para C-Level y comités de auditoría.

### Modelo de Datos Relacional:
* **Dim_Calendario:** Tabla de tiempo para cortes semanales y mensuales.
* **Dim_Squad:** Equipos de desarrollo, tech leads y áreas de negocio.
* **Fact_Jira_Issues:** Tickets, tipos de requerimiento (Feature, Bug, Tech Debt), Story Points, fechas de estado.
* **Fact_Incidentes_SLA:** Incidentes de producción, MTTR, severidad y porcentaje de disponibilidad.

### Medidas Clave en DAX (Ejemplos):

```dax
// 1. Predictibilidad de Entrega del Sprint
Sprint_Predictability = 
DIVIDE(
    CALCULATE(SUM(Fact_Jira_Issues[StoryPoints]), Fact_Jira_Issues[Estado] = "Done"),
    CALCULATE(SUM(Fact_Jira_Issues[StoryPoints]), Fact_Jira_Issues[Estado_Inicial] = "Committed"),
    0
)

// 2. Tiempo Medio de Recuperación (MTTR en Horas)
MTTR_Horas = 
AVERAGEX(
    FILTER(Fact_Incidentes_SLA, Fact_Incidentes_SLA[Severidad] IN {"SEV-1", "SEV-2"}),
    DATEDIFF(Fact_Incidentes_SLA[Fecha_Inicio], Fact_Incidentes_SLA[Fecha_Cierre], MINUTE) / 60.0
)

// 3. Balance de Deuda Técnica vs Nuevas Funcionalidades
Ratio_Deuda_Tecnica = 
DIVIDE(
    CALCULATE(COUNTROWS(Fact_Jira_Issues), Fact_Jira_Issues[Tipo] = "Tech Debt"),
    COUNTROWS(Fact_Jira_Issues),
    0
)
```

---

## 4. Reporte Ejecutivo Semanal (Status Report Template)

* **Resumen de Salud del Squad:** 🟢 *Verde / On Track* (Predictibilidad: 92%).
* **Entregas de Alto Impacto:** 3 microservicios desplegados sin afectación de SLAs.
* **Riesgos & Dependencias Identificadas:** Esperando aprobación de contrato de seguridad de API con aliado bancario (impacto estimado: 2 días de holgura disponible).
* **Acciones de Mitigación:** Reunión técnica de desbloqueo agendada para el miércoles; sin riesgo para el release de fin de mes.
