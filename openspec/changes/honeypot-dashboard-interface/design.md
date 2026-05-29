## Context

Actualmente, los honeypots Cowrie y Dionaea capturan eventos de ataque en tiempo real y los envían a n8n para procesamiento. Sin embargo, no existe una interfaz para que los equipos técnicos visualicen, analicen o testee estos eventos de forma interactiva. El sistema debe proporcionar:

1. Visibilidad en tiempo real de eventos capturados
2. Análisis de ataques mapeados a MITRE ATT&CK 
3. Capacidad de simular eventos para validar la pipeline
4. Control de acceso basado en autenticación JWT
5. Integración bidireccional con n8n

**Restricciones confirmadas:**
- JWT almacenado en .env
- MITRE ATT&CK hardcoded en migrations
- n8n: comunicación bidireccional (Express → n8n, n8n → Express)
- Stack confirmado: React + TailwindCSS + TanStack Table + Express + PostgreSQL
- Socket.IO para real-time con latencia < 100ms

## Goals / Non-Goals

**Goals:**
- Crear una interfaz web responsiva para visualizar eventos de honeypot
- Implementar sistema de autenticación JWT seguro
- Mapear eventos a técnicas MITRE ATT&CK con visualización de heatmaps
- Permitir simulación de ataques (reales vía n8n, fake en BD)
- Transmitir eventos en tiempo real vía WebSocket < 100ms
- Proporcionar métricas clave: MTTD, MTTR, counts, técnicas detectadas
- Garantizar ARQUITECTURA SOLID con separación de concerns

**Non-Goals:**
- Machine learning o análisis predictivo
- UI customizable por usuario
- Integración con otros honeypots (solo Cowrie + Dionaea)
- Multi-tenancy
- Almacenamiento de eventos por más de 90 días (no especificado en requisitos)

## Decisions

### 1. Arquitectura en 3 capas (SOLID principles)

**Decisión:** Frontend (React) ↔ Backend (Express API) ↔ Database (PostgreSQL)

**Rationale:**
- Separación de concerns clara
- Frontend desacoplado del backend permite cambios independientes
- API REST es agnóstica a UI (podría haber CLI, móvil, etc.)
- WebSocket y REST pueden coexistir sin conflictos

**Alternativas consideradas:**
- Monolito Next.js → Rechazado: No separa concerns, dificulta testing
- GraphQL → Rechazado: Complejidad innecesaria para estos datos

### 2. PostgreSQL como fuente de verdad

**Decisión:** Todos los eventos van a PostgreSQL primero, luego se broadcast vía WebSocket

**Rationale:**
- Garantiza persistencia de eventos
- Permite queries complejas (filtros por IP, técnica, fecha, etc.)
- WebSocket es para notificación, no almacenamiento
- n8n webhook escribe directamente en BD (vía Express endpoint)

**Alternativas consideradas:**
- Redis + WebSocket para real-time → Rechazado: Pérdida de datos si crashes
- Stream events directo a frontend → Rechazado: No hay audit trail

### 3. n8n bidireccional: Webhooks + HTTP calls

**Decisión:** 
- n8n ENVÍA eventos a Express POST /api/events (webhook)
- Express LLAMA n8n para trigger simulaciones POST /webhook/n8n/simulate

**Rationale:**
- n8n ya captura eventos, simplemente los pushea a Express
- Express puede invocar n8n workflows para simulaciones
- Fallback: si n8n no responde, simulate inserta evento fake en BD
- Mantiene n8n como orquestador de capturas, Express como API

**Alternativas consideradas:**
- Express polling n8n → Rechazado: Latencia innecesaria
- n8n como API principal → Rechazado: n8n no es diseñado para APIs REST scalables

### 4. JWT en .env con expiración

**Decisión:** 
- Secret JWT almacenado en .env
- Tokens con expiración (ej: 24h)
- Refresh token mechanism NO implementado (out of scope)

**Rationale:**
- Simple de configurar y desplegar
- Seguro si .env no se commitea
- Token en Authorization header (Bearer scheme)
- Validación de expiración en middleware

**Alternativas consideradas:**
- OAuth2 → Rechazado: Complejidad innecesaria para equipo técnico interno
- Session-based (cookies) → Rechazado: JWT es más escalable para múltiples instancias

### 5. WebSocket con Socket.IO para real-time

**Decisión:** 
- Socket.IO en mismo puerto que Express (3001) con namespace /socket.io
- Broadcast a todos los clientes conectados cuando evento llega a BD
- Latencia target: < 100ms

**Rationale:**
- Socket.IO maneja fallback a polling si WebSocket no disponible
- Integración simple con Express existente
- Broadcasting es O(n) clientes, suficiente para equipo técnico
- < 100ms es achievable con eventos en memoria + DB commit asincrónico

**Alternativas consideradas:**
- Server-Sent Events (SSE) → Rechazado: No es bidireccional, Socket.IO es mejor
- Polling REST → Rechazado: Mayor latencia y carga en servidor

### 6. Simulación con fallback

**Decisión:**
- POST /api/simulate con { type, mode: 'real' | 'simulated' }
- mode='real': Llama n8n webhook, espera respuesta
  - Si n8n falla (timeout 5s): Inserta evento fake como fallback
- mode='simulated': Inserta evento fake directo en BD
  - is_simulated=true para identificar en queries

**Rationale:**
- Permite testing incluso si n8n está down
- Datos fake para testing no contamina análisis
- Fallback automático mantiene sistema resiliente
- Transición suave: real → simulated si error

**Alternativas consideradas:**
- Solo simulación fake → Rechazado: No valida n8n integration
- Fallar si n8n no responde → Rechazado: Pobre UX para testing

### 7. MITRE ATT&CK hardcoded en migrations

**Decisión:** 
- 4 tablas:
  - mitre_tactic (matriz de 14 tácticas)
  - mitre_technique (700+ técnicas)
  - mitre_tactic_technique_mapping (relación M:N)
  - honeypot_event_technique_mapping (evento → técnicas detectadas)

**Rationale:**
- Evita llamadas externas a MITRE API
- Datos inmutables → caching perfecto
- Permite heatmaps y análisis eficientes
- Actualización anual en migrations si MITRE cambia

**Alternativas consideradas:**
- Llamar MITRE API en tiempo real → Rechazado: Latencia + dependencia externa
- Almacenar solo técnicas → Rechazado: Perderías contexto de tácticas

### 8. Estructura de tabla honeypot_events

**Decisión:**
- PK: id (UUID)
- Campos: timestamp, source_ip, destination_port, honeypot (cowrie|dionaea), 
  event_type (login_attempt|command_execution|file_access), 
  protocol, is_simulated, created_at
- Índices: (source_ip, timestamp), (event_type), (honeypot), (is_simulated)

**Rationale:**
- Timestamp es del evento, created_at es de inserción (audit trail)
- is_simulated permite filtrar fake events
- Índices para queries comunes (filtro por IP, técnica, fecha)
- UUID es portable, no secuencial

**Alternativas consideradas:**
- Timestamp como PK → Rechazado: No es único, UUID es mejor
- JSON blob para datos → Rechazado: Menos queryable

## Risks / Trade-offs

| Risk | Mitigation |
|------|-----------|
| **n8n indisponible durante simulación** | Fallback a evento fake automático, log de intento real |
| **WebSocket latencia > 100ms** | Eventos en buffer + batch broadcast si carga alta, monitoreo con métricas |
| **JWT secret compromised en .env** | Rotar secret en .env, invalidar tokens antiguos (requiere migration) |
| **Eventos duplicados desde n8n** | Idempotency key (hash timestamp + IP + tipo) en webhook |
| **Query performance en tabla grande** | Índices cuidadosos, limit 1000 eventos/query, archiving después 90 días |
| **Equipo confunde eventos fake con reales** | Badge visual "SIMULATED", filtro por defecto excluye fake, logs claros |
| **CORS vulnerabilidades** | Whitelist explícito de origins, no wildcard en prod |

## Migration Plan

**Fase 1: Database (Día 1)**
1. Ejecutar migrations PostgreSQL (9 tablas + índices)
2. Seed MITRE ATT&CK data desde CSV

**Fase 2: Express Backend (Día 1-2)**
1. Setup Express + TypeScript
2. Implement 15 endpoints (auth, events, metrics, mitre, simulate, webhook)
3. Middleware JWT + error handling
4. Socket.IO setup
5. Test con curl/Postman

**Fase 3: React Frontend (Día 2-3)**
1. Setup React + Vite + TailwindCSS
2. Login page (POST /auth/login)
3. Events table (TanStack Table)
4. Metrics dashboard
5. MITRE analysis (heatmap)
6. Simulate panel
7. WebSocket integration

**Fase 4: Integration + Testing (Día 3-4)**
1. n8n webhook configuration
2. Test evento real desde honeypot → n8n → Express → React
3. Test simulación → n8n → fallback
4. Load testing WebSocket
5. Docker Compose update

**Rollback:** Si algo falla en Fase 2-4, revert changes en Git. BD puede rollback con migration down.

## Open Questions

1. **Duración de almacenamiento eventos:** ¿90 días? ¿1 año? (Afecta archiving strategy)
2. **Capacidad esperada:** ¿100 eventos/día? ¿10k/día? (Afecta índices y batching)
3. **Número de usuarios concurrentes:** ¿5? ¿50? (Afecta WebSocket tuning)
4. **Actualización MITRE ATT&CK:** ¿Anual? ¿Bajo demanda? (Afecta migration strategy)
5. **Monitoreo/Alertas:** ¿Prometheus? ¿DataDog? ¿Solo logs? (Out of scope de este cambio)
