## ADDED Requirements

### Requirement: WebSocket connection to real-time event stream
The system SHALL establish WebSocket connections from React frontend to Express backend, enabling bidirectional communication for event streaming.

#### Scenario: Client connects to WebSocket
- **WHEN** React client initiates Socket.IO connection to ws://localhost:3001
- **THEN** WebSocket handshake succeeds, client receives connection confirmation with client_id

#### Scenario: Multiple clients can connect
- **WHEN** multiple browser tabs/users connect to WebSocket
- **THEN** system accepts all connections, maintains separate session for each client

#### Scenario: Connection persists during inactivity
- **WHEN** WebSocket client remains connected without sending messages
- **THEN** connection stays open indefinitely, heartbeat/ping keeps connection alive

#### Scenario: Reconnection on network failure
- **WHEN** network interruption causes temporary disconnect
- **THEN** Socket.IO client library automatically attempts reconnect (exponential backoff)

### Requirement: Broadcast events to all connected clients
The system SHALL broadcast new honeypot events to all connected WebSocket clients immediately when events are persisted to database.

#### Scenario: Event arrives from honeypot
- **WHEN** n8n sends new event to Express POST /api/events webhook
- **THEN** system stores event in PostgreSQL AND broadcasts to all connected clients via socket.emit('newEvent', {...})

#### Scenario: All clients receive event simultaneously
- **WHEN** event is broadcast
- **THEN** all connected WebSocket clients receive event notification within 100ms (latency target)

#### Scenario: Event message format
- **WHEN** event is broadcast
- **THEN** clients receive message with schema: { id, timestamp, source_ip, honeypot, event_type, protocol, risk_score, techniques: [...], is_simulated }

#### Scenario: Clients can filter events after receive
- **WHEN** client receives broadcast event
- **THEN** event data includes all fields to enable client-side filtering by IP, type, honeypot, etc.

### Requirement: Subscribe to specific event streams
The system SHALL allow clients to subscribe/unsubscribe from event streams based on filters (honeypot type, event type, IP range).

#### Scenario: Subscribe to specific honeypot
- **WHEN** client emits socket.emit('subscribe', { honeypot: 'cowrie' })
- **THEN** system subscribes client to cowrie events only, other clients still receive all events

#### Scenario: Subscribe to event type
- **WHEN** client emits socket.emit('subscribe', { eventType: 'login_attempt' })
- **THEN** system broadcasts only login_attempt events to this client

#### Scenario: Multiple subscriptions per client
- **WHEN** client emits multiple subscribe messages with different filters
- **THEN** system OR the conditions: send events matching any of the filters

#### Scenario: Unsubscribe from stream
- **WHEN** client emits socket.emit('unsubscribe', { honeypot: 'cowrie' })
- **THEN** system stops sending cowrie events to this client

### Requirement: Real-time latency guarantee
The system SHALL ensure event broadcast latency is less than 100ms from database insertion to WebSocket delivery.

#### Scenario: Measure broadcast latency
- **WHEN** event is inserted into PostgreSQL
- **THEN** all connected clients receive WebSocket event within 100ms

#### Scenario: Latency tracking
- **WHEN** event is broadcast
- **THEN** system includes timing metadata: db_timestamp, broadcast_timestamp, latency_ms in event message

#### Scenario: Handle high-frequency events
- **WHEN** system receives 1000+ events per minute
- **THEN** latency remains under 100ms, events are not dropped or queued indefinitely

### Requirement: Event acknowledgment mechanism
The system SHALL track client receipt of events to ensure delivery and provide metrics on delivery rates.

#### Scenario: Client acknowledges event receipt
- **WHEN** client receives event via socket.on('newEvent', ...)
- **THEN** client emits socket.emit('ack', { eventId, timestamp }) back to server

#### Scenario: Timeout for ack
- **WHEN** server broadcasts event and doesn't receive ack within 30 seconds
- **THEN** server logs unacknowledged event for monitoring (but doesn't retry)

#### Scenario: Delivery metrics
- **WHEN** tracking acknowledgments
- **THEN** system can report: total_events_broadcast, total_acks_received, delivery_rate_percentage

### Requirement: Handle WebSocket disconnection gracefully
The system SHALL detect client disconnections and clean up resources without affecting other connected clients.

#### Scenario: Client disconnects unexpectedly
- **WHEN** network failure causes abrupt disconnection
- **THEN** server detects connection loss, cleans up session data, other clients continue receiving events

#### Scenario: Client intentionally closes connection
- **WHEN** client closes browser tab or window
- **THEN** WebSocket disconnect is detected, server removes client from subscriptions

#### Scenario: Resume after reconnection
- **WHEN** client reconnects after temporary disconnection
- **THEN** new connection established, client can re-subscribe to event streams, missed events not replayed

### Requirement: Error handling for WebSocket communication
The system SHALL handle and communicate errors that occur during WebSocket operations.

#### Scenario: Subscribe to invalid filter
- **WHEN** client emits socket.emit('subscribe', { invalidField: 'value' })
- **THEN** server returns error via socket.emit('error', { message: "Invalid filter field" })

#### Scenario: Authentication failure on WebSocket
- **WHEN** client attempts WebSocket connection without valid JWT
- **THEN** server rejects connection with 401 Unauthorized before WebSocket established

#### Scenario: Broadcast event error
- **WHEN** error occurs during broadcast (e.g., client message queue full)
- **THEN** server logs error, continues broadcasting to other clients, does not crash

### Requirement: Heartbeat/keepalive for long connections
The system SHALL send periodic heartbeat messages to keep idle WebSocket connections alive and detect dead connections.

#### Scenario: Server sends heartbeat
- **WHEN** no messages exchanged for 30 seconds
- **THEN** server sends ping frame, client responds with pong (Socket.IO handles automatically)

#### Scenario: Dead connection detection
- **WHEN** client fails to respond to ping within 5 seconds
- **THEN** server closes connection and cleans up client session
