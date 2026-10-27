# 📋 Plantilla Estándar de Historia de Usuario (INVEST + BDD Gherkin)

> **Autor:** Rosmar Mendoza — Ingeniera de Sistemas & Technical Project Manager  
> **Framework:** Agile / Scrum & BDD (Behavior-Driven Development)

---

## 1. Principio INVEST de Validación

Antes de que una historia entre al Sprint Backlog, debe cumplir los criterios **INVEST**:

| Criterio | Descripción | Estado de Verificación |
| :--- | :--- | :---: |
| **I - Independent** | La historia se puede desarrollar y desplegar sin depender estrictamente de otra historia en el mismo sprint. | ✅ |
| **N - Negotiable** | El alcance de la solución técnica es negociable entre el Product Owner y el equipo de ingeniería. | ✅ |
| **V - Valuable** | Aporta valor tangible directo al usuario final o a la estabilidad del negocio. | ✅ |
| **E - Estimable** | El equipo cuenta con suficiente contexto para estimar el esfuerzo en Story Points (Fibonacci: 1, 2, 3, 5, 8). | ✅ |
| **S - Small** | Diseñada para completarse dentro de una fracción del Sprint (no excede 3-5 días de desarrollo). | ✅ |
| **T - Testable** | Contiene criterios de aceptación objetivos y verificables por QA automatizado o manual. | ✅ |

---

## 2. Estructura Canónica de la Historia de Usuario

### 📌 Identificador: `[PROJ-1042] Despliegue de Endpoint de Consulta de Balance en Tiempo Real`
* **Épica:** *Gestión de Crédito Digital y Microservicios Fintech*
* **Prioridad:** *Alta (Bloqueante para release Q1)*
* **Estimación:** *5 Story Points*
* **Squad Responsable:** *Core Payments & APIs*

### 📝 Narrativa de Valor
```gherkin
Como: Usuario registrado de la billetera digital
Quiero: Consultar el balance disponible de mi línea de crédito en tiempo real
Para: Tomar decisiones informadas antes de autorizar una transacción de compra.
```

---

## 3. Criterios de Aceptación (Formato Gherkin / BDD)

### Escenario 1: Consulta exitosa con fondos disponibles
```gherkin
Given que el usuario está autenticado con un token JWT válido
And tiene una línea de crédito activa con un límite asignado de $500.000 COP
When realiza una petición HTTP GET al endpoint `/v1/credit/balance`
Then el servicio debe responder con código HTTP 200 (OK)
And la respuesta debe incluir el balance disponible, el monto utilizado y la fecha de corte
And el tiempo de respuesta (P95) debe ser inferior a 150 milisegundos.
```

### Escenario 2: Usuario con línea de crédito bloqueada o en mora
```gherkin
Given que el usuario está autenticado
And su estado de cuenta registra mora mayor a 30 días
When realiza una petición HTTP GET al endpoint `/v1/credit/balance`
Then el servicio debe responder con código HTTP 403 (Forbidden)
And el cuerpo de respuesta debe contener el código de error `CREDIT_LINE_LOCKED` y el mensaje de contacto a soporte.
```

### Escenario 3: Falla temporal de conexión con el Core Bancario (Resiliencia)
```gherkin
Given que el microservicio bancario aguas abajo responde con timeout (>2000 ms)
When el usuario consulta su balance
Then el sistema debe activar el patrón Circuit Breaker
And retornar un mensaje controlado HTTP 503 con Retry-After
And registrar la traza del error en Datadog con tags de severidad Warning para observabilidad.
```

---

## 4. Criterios de Preparación y Finalización (DoR vs. DoD)

### 🚪 Definition of Ready (DoR) — Entrada al Sprint
- [x] La historia cumple con el principio INVEST.
- [x] Criterios de aceptación detallados y validados con el Product Owner y Tech Lead.
- [x] Contrato OpenAPI / Swagger de la API definido y revisado.
- [x] Dependencias externas identificadas y resueltas.
- [x] Estimación aprobada por el squad en la sesión de Planning.

### ✅ Definition of Done (DoD) — Salida y Despliegue
- [x] Código implementado siguiendo estándares de arquitectura limpia y linters.
- [x] Cobertura de pruebas unitarias superior al **85%**.
- [x] Pruebas de integración de API ejecutadas exitosamente en entorno Staging.
- [x] Métricas de telemetría y logs configurados en Datadog para monitoreo de SLA.
- [x] Pull Request aprobado por al menos 2 revisores (Tech Lead y Peer Reviewer).
- [x] Pipeline CI/CD ejecutado sin errores y documentación de API actualizada en Confluence.
