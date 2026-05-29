## Why

El sistema de honeypots (Cowrie + Dionaea) captura eventos de ataque en tiempo real, pero actualmente no existe una interfaz para visualizar, analizar o testear estos eventos. Los equipos técnicos necesitan una forma intuitiva de acceder a los datos, mapear ataques a MITRE ATT&CK, y simular eventos para validar la pipeline de detección. Esto es crítico para evaluar la efectividad del honeypot y la respuesta de n8n.

## What Changes

- **Backend API REST**: Nuevos endpoints en Express para eventos, métricas, autenticación y simulación
- **Base de datos**: 9 nuevas tablas PostgreSQL (honeypot_events, users, mitre_techniques, attack_simulations, etc.)
- **Frontend React**: Aplicación web con login, dashboard de eventos, análisis MITRE y panel de simulación
- **Autenticación JWT**: Sistema de login con tokens JWT almacenados en .env
- **Integración n8n**: Webhooks bidireccionales para trigger eventos simulados y recibir eventos reales
- **WebSocket Real-time**: Socket.IO para broadcast de eventos en vivo a < 100ms
- **Simulación de ataques**: Capacidad de generar eventos fake (IPs aleatorias, timestamps actuales, técnicas aleatorias) o reales vía n8n

## Capabilities

### New Capabilities
- `event-management`: Consulta, filtrado, búsqueda y paginación de eventos capturados por honeypots
- `mitre-analysis`: Mapeo de eventos a MITRE ATT&CK, heatmaps tácticas × técnicas, cobertura
- `attack-simulation`: Simulación de eventos reales (vía n8n) o fake (en BD) para testing y validación
- `user-authentication`: Sistema de login con JWT, almacenamiento seguro de credenciales
- `real-time-events`: Entrega de eventos vía WebSocket con latencia < 100ms
- `metrics-dashboard`: MTTD, MTTR, counts de eventos, técnicas detectadas, IPs únicas
- `api-backend`: REST API en Express con validación JWT, manejo de errores, logging

### Modified Capabilities
<!-- No existing capabilities are being modified - all are new for this change -->

## Impact

- **PostgreSQL**: Nuevas 9 tablas en BD (honeypot_events, users, mitre_attack_techniques, mitre_tactic_technique_mapping, user_sessions, attack_simulations, event_enrichment, event_webhooks, system_logs)
- **n8n**: Nuevos webhooks para recibir eventos y trigger simulaciones
- **Express**: Nuevos servicios en localhost:3001 (API REST + WebSocket)
- **React**: Nueva SPA en localhost:3000 con React + TailwindCSS + TanStack Table
- **Dependencias**: jsonwebtoken, bcryptjs, socket.io, socket.io-client, @tanstack/react-table, tailwindcss
- **Arquitectura**: Flujo Honeypots → n8n → Express (webhook) → React (WebSocket + REST)
- **Seguridad**: JWT en headers Authorization, CORS habilitado, validación de entrada, hashing de contraseñas
