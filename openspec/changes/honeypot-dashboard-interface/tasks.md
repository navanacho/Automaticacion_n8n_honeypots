## 1. Database Setup & Migrations

- [ ] 1.1 Create PostgreSQL migration file for users table (id, username, password_hash, created_at, updated_at)
- [ ] 1.2 Create migration for honeypot_events table (id, timestamp, source_ip, destination_port, honeypot, event_type, protocol, risk_score, is_simulated, created_at, indexes on ip/type/honeypot/simulated)
- [ ] 1.3 Create migration for mitre_tactic table (id, tactic_id: "TA0001", name, description)
- [ ] 1.4 Create migration for mitre_technique table (id, technique_id: "T1110", name, description, subtechniques)
- [ ] 1.5 Create migration for mitre_tactic_technique_mapping table (tactic_id FK, technique_id FK, composite PK)
- [ ] 1.6 Create migration for honeypot_event_technique_mapping table (event_id FK, technique_id FK, confidence_score)
- [ ] 1.7 Create migration for user_sessions table (id, user_id FK, login_timestamp, ip_address, user_agent)
- [ ] 1.8 Create migration for attack_simulations table (id, user_id FK, timestamp_requested, mode, type, status, event_id FK, n8n_response, fallback_reason)
- [ ] 1.9 Create migration for system_logs table (id, timestamp, level, message, context JSON, ip_address)
- [ ] 1.10 Run all migrations and verify schema
- [ ] 1.11 Seed MITRE ATT&CK data from CSV (14 tactics + 700+ techniques) into mitre_tactic and mitre_technique tables
- [ ] 1.12 Seed initial test user (username: admin, password: hashed) for login testing
- [ ] 1.13 Create database indexes for query performance (composite indexes on frequently filtered columns)

## 2. Express Backend Setup & Core API

- [ ] 2.1 Initialize Express.js project with TypeScript, package.json dependencies (express, typescript, dotenv, pg, jsonwebtoken, bcryptjs, socket.io, cors)
- [ ] 2.2 Create .env.example with required variables (DATABASE_URL, JWT_SECRET, NODE_ENV, PORT, N8N_WEBHOOK_URL, CORS_ORIGIN)
- [ ] 2.3 Create main Express app initialization file (app.ts) with middleware setup
- [ ] 2.4 Implement JWT token generation utility (generate token with userId, username, iat, exp)
- [ ] 2.5 Implement JWT validation middleware (extract token from Authorization header, verify signature & expiration, attach req.user)
- [ ] 2.6 Implement global error handling middleware (catch errors, log, return consistent error format)
- [ ] 2.7 Implement request logging middleware (log method, path, status, response_time_ms to console + file)
- [ ] 2.8 Implement CORS middleware (allow localhost:3000 in dev, whitelist from .env in prod)
- [ ] 2.9 Setup PostgreSQL connection pool (pg Client, create connection manager)
- [ ] 2.10 Create utility functions for common database queries (findUserByUsername, findEventById, queryEvents with filters)

## 3. Authentication Endpoints & User Management

- [ ] 3.1 Implement POST /api/auth/login (accept username/password, verify bcrypt, return JWT token + user object)
- [ ] 3.2 Implement password hashing utility using bcryptjs (cost factor ≥10)
- [ ] 3.3 Implement password verification utility (compare plaintext against bcrypt hash)
- [ ] 3.4 Implement GET /api/auth/verify (validate JWT token, return user info if valid)
- [ ] 3.5 Implement token expiration (24 hour expiry, verify exp claim in middleware)
- [ ] 3.6 Implement failed login attempt logging (record failed attempt with timestamp, ip_address, username to system_logs)
- [ ] 3.7 Implement successful login session tracking (record to user_sessions table with login_timestamp, ip_address, user_agent)
- [ ] 3.8 Add Bearer token extraction from Authorization header (case-insensitive)
- [ ] 3.9 Test login endpoint with curl/Postman (valid credentials, wrong password, missing fields)

## 4. Event Management Endpoints

- [ ] 4.1 Implement GET /api/events (return paginated events list with default limit=50, support pagination metadata)
- [ ] 4.2 Implement query parameter filtering on GET /api/events (support: ip, eventType, honeypot, startDate, endDate, includeSimulated)
- [ ] 4.3 Implement full-text search GET /api/events/search?q=<query> (search across IP, event_type, protocol)
- [ ] 4.4 Implement GET /api/events/{eventId} (return single event with all fields + mapped techniques)
- [ ] 4.5 Implement POST /api/webhooks/events (receive events from n8n, validate required fields, store in DB, broadcast via WebSocket)
- [ ] 4.6 Implement idempotency key checking for webhook (detect duplicate events by hash of timestamp+ip+type, return 200 but no duplicate)
- [ ] 4.7 Implement event-to-technique mapping on event creation (analyze event_type, map to MITRE techniques, store in mapping table)
- [ ] 4.8 Implement event risk_score calculation (assign 1-10 based on event_type and mapped techniques severity)
- [ ] 4.9 Apply JWT middleware to all event endpoints except webhook
- [ ] 4.10 Test event endpoints (POST from n8n, GET with various filters, search, single event retrieval)

## 5. MITRE Analysis Endpoints

- [ ] 5.1 Implement GET /api/mitre/tactics (return all 14 tactics with id, name, description)
- [ ] 5.2 Implement GET /api/mitre/techniques (return all techniques with id, name, description, related tactics)
- [ ] 5.3 Implement GET /api/mitre/techniques/{techniqueId} (return single technique details + related tactics)
- [ ] 5.4 Implement GET /api/mitre/tactics/{tacticId}/techniques (return all techniques for specific tactic)
- [ ] 5.5 Implement GET /api/mitre/heatmap (generate tactic×technique matrix with event counts, support period filter)
- [ ] 5.6 Implement heatmap data caching (calculate once per 5 minutes, cache in memory or Redis)
- [ ] 5.7 Implement GET /api/mitre/coverage (return total_techniques_detected, coverage_percentage, top_tactics, top_techniques)
- [ ] 5.8 Implement GET /api/mitre/techniques/prevalence (return top N techniques sorted by event count)
- [ ] 5.9 Apply JWT middleware to all MITRE endpoints
- [ ] 5.10 Test MITRE endpoints (verify tactics/techniques data, heatmap format, coverage calculation)

## 6. Metrics & Dashboard Endpoints

- [ ] 6.1 Implement GET /api/metrics/events (return total_count, unique_ips, event_type_distribution)
- [ ] 6.2 Implement GET /api/metrics/mttd (calculate mean time to detection, support period + filter parameters)
- [ ] 6.3 Implement GET /api/metrics/mttr (calculate mean time to resolution for resolved events)
- [ ] 6.4 Implement GET /api/metrics/dashboard (consolidated endpoint returning MTTD, MTTR, counts, coverage, top_techniques, trend_24h)
- [ ] 6.5 Implement trend calculation (hourly event counts for last 24 hours for dashboard graph)
- [ ] 6.6 Add Cache-Control headers to metrics endpoints (max-age=300 seconds)
- [ ] 6.7 Implement custom metric queries (support combining filters: ip, technique, honeypot, date range, excludeSimulated)
- [ ] 6.8 Implement metrics data formatting (times in seconds, percentages as 0.0-1.0, timestamps in ISO 8601)
- [ ] 6.9 Apply JWT middleware to all metrics endpoints
- [ ] 6.10 Test metrics endpoints (verify calculations, filtering, caching, response format)

## 7. Attack Simulation Endpoints

- [ ] 7.1 Implement POST /api/simulate (accept type, mode params, validate inputs, route to real or fake simulation)
- [ ] 7.2 Implement real simulation flow (call n8n webhook, wait 5s timeout, handle success/failure/timeout)
- [ ] 7.3 Implement automatic fallback to fake event (if n8n fails/times out, insert fake event, log failure reason)
- [ ] 7.4 Implement fake simulation flow (generate random IP from private range, random technique, current timestamp, is_simulated=true)
- [ ] 7.5 Implement idempotency for simulations (accept optional idempotencyKey, check if processed in last 24h, return cached result or new event)
- [ ] 7.6 Implement simulation record creation (store in attack_simulations table with mode, status, user_id, n8n_response)
- [ ] 7.7 Implement GET /api/simulations (return paginated list of past simulations, sorted by timestamp desc)
- [ ] 7.8 Implement simulation response format (return 201 Created or 202 Accepted with eventId, mode, status, created_event details)
- [ ] 7.9 Apply JWT middleware to simulate endpoints
- [ ] 7.10 Test simulation endpoints (real mode with n8n, fake mode, fallback behavior, idempotency)

## 8. WebSocket Real-Time Events Setup

- [ ] 8.1 Install socket.io dependency (npm install socket.io)
- [ ] 8.2 Create Socket.IO server instance attached to Express app on same port (3001)
- [ ] 8.3 Implement WebSocket connection handler (accept connection, assign client_id, send connection confirmation)
- [ ] 8.4 Implement event broadcast to all connected clients (when event stored in DB, emit newEvent to all clients)
- [ ] 8.5 Implement event subscription/filtering (clients can emit subscribe message with filters, store per-client filter preferences)
- [ ] 8.6 Implement selective broadcast (only send events matching client's subscribed filters)
- [ ] 8.7 Implement client acknowledgment mechanism (track acks, report delivery metrics)
- [ ] 8.8 Implement heartbeat/keepalive (send ping every 30s, detect dead connections)
- [ ] 8.9 Implement disconnection cleanup (remove client from active connections, clean up subscriptions)
- [ ] 8.10 Implement WebSocket error handling (send error messages for invalid subscriptions, log errors)
- [ ] 8.11 Implement broadcast latency tracking (include db_timestamp, broadcast_timestamp, latency_ms in event messages)
- [ ] 8.12 Test WebSocket connections (verify broadcast latency < 100ms, multiple clients, reconnection)

## 9. React Frontend Setup & Authentication

- [ ] 9.1 Initialize React + Vite project with TypeScript, TailwindCSS, React Router
- [ ] 9.2 Create .env file with API_URL=http://localhost:3001
- [ ] 9.3 Create authentication context (store JWT token, user info, provide login/logout functions)
- [ ] 9.4 Create JWT token storage utility (store in localStorage, retrieve, validate expiration)
- [ ] 9.5 Implement login page component (form with username/password, POST to /api/auth/login, store token on success)
- [ ] 9.6 Implement token validation on app load (check if token exists and not expired, redirect to login if invalid)
- [ ] 9.7 Create HTTP client wrapper (attach JWT to Authorization header, handle 401 responses)
- [ ] 9.8 Implement protected route component (redirect to login if no valid token)
- [ ] 9.9 Create app layout (navigation, header with user info, logout button, main content area)
- [ ] 9.10 Test login flow (valid credentials, wrong password, token storage, redirect to protected routes)

## 10. Events Table & Filters UI

- [ ] 10.1 Install TanStack React Table and dependencies
- [ ] 10.2 Create Events table component (display: timestamp, source_ip, honeypot, event_type, protocol, risk_score, techniques)
- [ ] 10.3 Implement TanStack Table with pagination (default 50 items per page, next/prev buttons)
- [ ] 10.4 Implement column sorting (click column header to sort ascending/descending)
- [ ] 10.5 Implement filter panel (dropdowns/inputs for: ip, eventType, honeypot, date range, includeSimulated)
- [ ] 10.6 Implement search box (full-text search via GET /api/events/search)
- [ ] 10.7 Implement filter application (update table on filter change, show active filter count)
- [ ] 10.8 Implement row expansion (click row to see full event details + mapped techniques)
- [ ] 10.9 Implement event refresh button (manual refresh + auto-refresh toggle every 30s)
- [ ] 10.10 Style events table with TailwindCSS (responsive, hover effects, visual hierarchy)
- [ ] 10.11 Test events table (pagination, sorting, filtering, search, row expansion)

## 11. MITRE Analysis Dashboard UI

- [ ] 11.1 Create MITRE analysis panel component
- [ ] 11.2 Implement heatmap visualization (fetch /api/mitre/heatmap, display tactic×technique matrix with color gradient by event count)
- [ ] 11.3 Implement heatmap interactivity (hover to see technique details, click to filter events by technique)
- [ ] 11.4 Create coverage summary component (display total_techniques_detected, coverage_percentage, visual progress bar)
- [ ] 11.5 Create top techniques component (display top N techniques with event counts, sortable list)
- [ ] 11.6 Create top tactics component (display top N tactics with event counts, sortable list)
- [ ] 11.7 Implement filters for MITRE analysis (period: 24h/7d/30d, honeypot filter, excludeSimulated toggle)
- [ ] 11.8 Implement data refresh for MITRE panel (fetch latest data on filter change, cache for 5 minutes)
- [ ] 11.9 Style MITRE components with TailwindCSS (responsive, readable heatmap, clear legends)
- [ ] 11.10 Test MITRE analysis (verify heatmap correctness, coverage calculations, filter application)

## 12. Metrics Dashboard UI

- [ ] 12.1 Create metrics dashboard component (main summary panel on homepage)
- [ ] 12.2 Implement MTTD card (display mean time to detection in human readable format)
- [ ] 12.3 Implement MTTR card (display mean time to resolution with visual indicator)
- [ ] 12.4 Implement event count cards (total_events, real_events, simulated_events with trend indicators)
- [ ] 12.5 Implement unique IPs card (display count, clickable to see IP list)
- [ ] 12.6 Implement 24-hour event trend chart (line/bar chart of hourly counts)
- [ ] 12.7 Implement event type distribution chart (pie/doughnut chart showing breakdown)
- [ ] 12.8 Implement honeypot distribution chart (bar chart showing cowrie vs dionaea)
- [ ] 12.9 Create metrics refresh mechanism (fetch /api/metrics/dashboard every 30s, with loading state)
- [ ] 12.10 Style metrics dashboard with TailwindCSS (responsive grid, color-coded cards, readable charts)
- [ ] 12.11 Test metrics dashboard (verify chart accuracy, auto-refresh, responsive on mobile)

## 13. Attack Simulation UI

- [ ] 13.1 Create simulation panel component
- [ ] 13.2 Implement simulation type selector (radio buttons: brute_force, command_execution, file_access, random)
- [ ] 13.3 Implement mode selector (radio buttons: real, simulated)
- [ ] 13.4 Implement mode description text (explain real vs simulated, n8n integration for real)
- [ ] 13.5 Create simulate button (POST to /api/simulate with selected params)
- [ ] 13.6 Implement response display (show created event details, status, latency)
- [ ] 13.7 Create simulation history list (fetch /api/simulations, display past simulations with status badge)
- [ ] 13.8 Implement error display (show n8n failure reasons, fallback notifications)
- [ ] 13.9 Add loading state (show spinner during simulation execution)
- [ ] 13.10 Style simulation panel with TailwindCSS (clear form layout, visual feedback)
- [ ] 13.11 Test simulation UI (real mode success/failure, fake mode, error handling)

## 14. WebSocket Client Integration

- [ ] 14.1 Install socket.io-client dependency
- [ ] 14.2 Create WebSocket connection hook (useSocket: establish connection on mount, store client reference)
- [ ] 14.3 Implement connection state tracking (isConnected boolean, show connection indicator)
- [ ] 14.4 Implement newEvent listener (listen for newEvent, update events table in real-time)
- [ ] 14.5 Implement reconnection handling (auto-reconnect on disconnect, show reconnecting indicator)
- [ ] 14.6 Implement subscription management (allow user to filter real-time events by honeypot/type)
- [ ] 14.7 Implement event acknowledgment (send ack back to server when event received)
- [ ] 14.8 Add real-time event badge (show "New events" with count when events arrive)
- [ ] 14.9 Implement optional event notification (browser notification on new critical event)
- [ ] 14.10 Test WebSocket integration (verify real-time delivery, latency, reconnection)

## 15. Navigation & Page Layout

- [ ] 15.1 Create main navigation component (nav bar with links: Events, Metrics, MITRE Analysis, Simulations, User Profile)
- [ ] 15.2 Create Events page (full-width events table with filters)
- [ ] 15.3 Create Dashboard/Metrics page (metrics cards + charts)
- [ ] 15.4 Create MITRE Analysis page (heatmap + coverage + top techniques)
- [ ] 15.5 Create Simulation page (simulation form + history table)
- [ ] 15.6 Create User Profile page (display current user info, logout button)
- [ ] 15.7 Create 404 page (not found route)
- [ ] 15.8 Implement routing with React Router (set up routes for all pages)
- [ ] 15.9 Style navigation with TailwindCSS (responsive, active link indicator, mobile hamburger menu)
- [ ] 15.10 Test page navigation (all links work, routing is clean, protected routes redirect)

## 16. Error Handling & User Feedback

- [ ] 16.1 Implement toast/notification system (success, error, warning, info messages)
- [ ] 16.2 Create error boundary component (catch React errors, display fallback UI)
- [ ] 16.3 Implement loading states (show spinners on async operations)
- [ ] 16.4 Implement API error handling (catch 4xx/5xx responses, show user-friendly error messages)
- [ ] 16.5 Create empty states (show helpful messages when no events, no simulations, etc.)
- [ ] 16.6 Implement request timeouts (show timeout message if request takes >30s)
- [ ] 16.7 Add input validation feedback (show field errors on login form, simulation form)
- [ ] 16.8 Implement connection loss recovery (show offline indicator, retry button, queue actions)
- [ ] 16.9 Test error scenarios (invalid API responses, network failures, malformed data)

## 17. Testing & QA Backend

- [ ] 17.1 Create unit tests for JWT utilities (token generation, validation, expiration)
- [ ] 17.2 Create unit tests for password hashing (bcrypt generation, verification)
- [ ] 17.3 Create integration tests for auth endpoints (login success/failure, invalid input)
- [ ] 17.4 Create integration tests for event endpoints (POST webhook, GET with filters, search)
- [ ] 17.5 Create integration tests for MITRE endpoints (verify heatmap, coverage, prevalence calculations)
- [ ] 17.6 Create integration tests for metrics endpoints (verify MTTD, MTTR, counts calculations)
- [ ] 17.7 Create integration tests for simulation endpoints (real mode, fake mode, fallback, idempotency)
- [ ] 17.8 Create load test for WebSocket (broadcast 1000+ events/min, verify < 100ms latency)
- [ ] 17.9 Create API documentation (README with endpoint descriptions, curl examples)
- [ ] 17.10 Run full backend test suite, fix any failures

## 18. Testing & QA Frontend

- [ ] 18.1 Create component tests for login page (successful login, error handling, form validation)
- [ ] 18.2 Create component tests for events table (pagination, sorting, filtering, search)
- [ ] 18.3 Create component tests for metrics dashboard (chart rendering, auto-refresh)
- [ ] 18.4 Create component tests for MITRE analysis (heatmap rendering, interactivity)
- [ ] 18.5 Create component tests for simulation panel (form submission, response display)
- [ ] 18.6 Create integration tests for WebSocket (connection, disconnect, real-time updates)
- [ ] 18.7 Test responsive design (mobile, tablet, desktop viewports)
- [ ] 18.8 Test browser compatibility (Chrome, Firefox, Safari)
- [ ] 18.9 Create user acceptance test checklist (end-to-end scenarios)
- [ ] 18.10 Run full frontend test suite, fix any failures

## 19. n8n Integration & Testing

- [ ] 19.1 Configure n8n webhook endpoint (point to POST /api/webhooks/events on Express)
- [ ] 19.2 Create n8n test workflow (generate sample honeypot event, send to Express webhook)
- [ ] 19.3 Test real simulation flow (simulate → n8n → event created in DB → broadcast to React)
- [ ] 19.4 Test fallback logic (n8n timeout → fake event created automatically)
- [ ] 19.5 Implement n8n error logging (log n8n response codes, timeout reasons)
- [ ] 19.6 Test idempotency (send duplicate events, verify no duplicates in DB)
- [ ] 19.7 Test webhook validation (send malformed events, verify 400 responses)
- [ ] 19.8 Document n8n integration (webhook URL, expected payload format, response format)
- [ ] 19.9 End-to-end test (real honeypot events → n8n → Express → React dashboard)

## 20. Docker & Deployment

- [ ] 20.1 Create Docker Compose file (PostgreSQL, Express backend, React frontend services)
- [ ] 20.2 Create .dockerignore files for frontend and backend
- [ ] 20.3 Create Dockerfile for Express backend (multi-stage build, node base)
- [ ] 20.4 Create Dockerfile for React frontend (build with Vite, serve with nginx)
- [ ] 20.5 Create docker-compose.yml with environment variables, volume mappings
- [ ] 20.6 Update .env.example with all required environment variables
- [ ] 20.7 Create database initialization script (run migrations on compose up)
- [ ] 20.8 Test Docker Compose setup (services start, logs are clean, ports are accessible)
- [ ] 20.9 Create deployment documentation (how to build, run, troubleshoot)
- [ ] 20.10 Create GitHub Actions workflow for automated testing (optional)

## 21. Documentation & Handoff

- [ ] 21.1 Create API documentation (OpenAPI/Swagger spec or README with all endpoints)
- [ ] 21.2 Create database schema diagram (visual representation of 9 tables)
- [ ] 21.3 Create architecture diagram (Honeypots → n8n → Express → React flow)
- [ ] 21.4 Create user guide (how to login, navigate, run simulations, read dashboards)
- [ ] 21.5 Create developer guide (project setup, running locally, extending features)
- [ ] 21.6 Create troubleshooting guide (common issues, solutions, logs to check)
- [ ] 21.7 Create MITRE mapping reference (how events map to techniques)
- [ ] 21.8 Document WebSocket event schema (event structure, field meanings)
- [ ] 21.9 Create n8n integration guide (webhook setup, test payload examples)
- [ ] 21.10 Review all documentation for clarity and completeness

## 22. Performance & Optimization

- [ ] 22.1 Add database query indexes (ensure composite indexes on frequently queried columns)
- [ ] 22.2 Implement query result caching (Redis or in-memory cache for MITRE data, heatmap)
- [ ] 22.3 Optimize WebSocket broadcasts (batch events if > 100/second, implement backpressure)
- [ ] 22.4 Lazy load React components (code-splitting for pages)
- [ ] 22.5 Optimize images/assets (compress, use appropriate formats)
- [ ] 22.6 Implement request deduplication (frontend: avoid duplicate API calls)
- [ ] 22.7 Monitor performance metrics (database query times, API response times, WebSocket latency)
- [ ] 22.8 Load test the system (simulate concurrent users, high event volume)
- [ ] 22.9 Identify and fix performance bottlenecks
- [ ] 22.10 Document performance baseline (expected response times, capacity limits)

## 23. Security Hardening

- [ ] 23.1 Review JWT implementation (ensure secret is strong, tokens not exposed in logs)
- [ ] 23.2 Implement rate limiting on sensitive endpoints (login, simulate)
- [ ] 23.3 Add input validation on all endpoints (reject suspicious payloads)
- [ ] 23.4 Implement SQL injection prevention (use parameterized queries everywhere)
- [ ] 23.5 Review CORS configuration (whitelist only necessary origins)
- [ ] 23.6 Implement Content Security Policy headers
- [ ] 23.7 Review environment variable security (.env not in git, secrets rotated)
- [ ] 23.8 Add HTTPS/TLS support (SSL certificates in production)
- [ ] 23.9 Document security best practices (password policies, token rotation, audit logging)
- [ ] 23.10 Conduct security review (check for common vulnerabilities, OWASP Top 10)

## 24. Final Integration & Launch

- [ ] 24.1 Full end-to-end test (login → view events → run simulation → see metrics → real-time updates)
- [ ] 24.2 Verify all 15+ API endpoints are working
- [ ] 24.3 Verify all React pages are functional and responsive
- [ ] 24.4 Verify WebSocket latency < 100ms under load
- [ ] 24.5 Verify n8n integration (real events flowing through)
- [ ] 24.6 Verify JWT authentication on all protected routes
- [ ] 24.7 Verify database schema and migrations are clean
- [ ] 24.8 Run final test suite (all unit, integration, end-to-end tests pass)
- [ ] 24.9 Create release notes (summary of features, known issues, future work)
- [ ] 24.10 Deploy to staging environment and validate
- [ ] 24.11 Get team approval and sign-off
- [ ] 24.12 Deploy to production
