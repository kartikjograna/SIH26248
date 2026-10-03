# System Architecture: Immersive Multi-Domain Decision-Making Trainer

## 1. High-Level Architecture
The system follows a **Client-Server Architecture** utilizing a real-time event-driven model.

### Architecture Diagram (Conceptual)
`[User Clients]` $\leftrightarrow$ `[Socket.io Gateway]` $\leftrightarrow$ `[Chaos Middleware]` $\leftrightarrow$ `[Scenario Engine]` $\leftrightarrow$ `[State Store/DB]`

## 2. Component Breakdown

### A. Frontend (The Interface)
- **Tech Stack**: React, Tailwind CSS, Three.js (for 3D Map) / Leaflet (for 2D Map).
- **Modules**:
    - **Tactical Display**: A spatial map showing unit positions and events.
    - **Comms Feed**: A chat/report interface that displays incoming (potentially degraded) messages.
    - **Role Manager**: Handles permissions based on user role (Commander vs. Field Op).

### B. Backend (The Orchestrator)
- **Tech Stack**: Node.js, Express, Socket.io.
- **Core Services**:
    - **Session Manager**: Handles multiplayer lobby, role assignment, and synchronization.
    - **Event Dispatcher**: Manages the "Ground Truth" state of the scenario.
    - **AAR Logger**: Captures every single state change and user action into the database for later analysis.

### C. The Chaos Middleware (The Interceptor)
This is the critical layer that sits between the **Event Dispatcher** and the **User Clients**.
- **Logic Flow**: 
    1. `Scenario Engine` $\rightarrow$ `Event: Enemy Spotted at X`
    2. `Chaos Middleware` $\rightarrow$ *Checks current noise level* $\rightarrow$ *Applies Delay (10s)* $\rightarrow$ *Rolls for Dropout (15% chance)*.
    3. `Socket.io` $\rightarrow$ `Client: Enemy Spotted at X` (delivered 10s later).

### D. Database Layer
- **Tech Stack**: MongoDB.
- **Collections**:
    - `Scenarios`: JSON definitions of events, triggers, and domains.
    - `Sessions`: Real-time state of active training exercises.
    - `DecisionLogs`: Every action taken, linked to the specific information available to the user at that millisecond.

## 3. Data Flow: The Decision Loop
1. **Truth Generation**: The Scenario Engine generates a world event (e.g., "Air Raid started").
2. **Information Filtering**: The Chaos Middleware filters this event based on the instructor's settings and the user's role.
3. **Perception**: The User sees the (possibly degraded) event on their map/feed.
4. **Action**: The User submits a decision (e.g., "Move to Cover").
5. **Logging**: The AAR Logger records: `[Time] | [Truth: Air Raid] | [Perception: No Info] | [Action: Move to Cover]`.

## 4. Scalability and Constraints
- **Real-time Sync**: Socket.io ensures low-latency updates for multiplayer coordination.
- **Modularity**: The Chaos Middleware is decoupled from the Scenario Engine, allowing new types of degradation (e.g., "packet corruption") to be added without changing the core simulation.
- **Persistence**: MongoDB allows for the storage of complex, nested scenario scripts.
