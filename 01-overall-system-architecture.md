# 🏗️ Overall System Architecture

This flowchart shows the complete communication flow between all components in the Fleet Tracker PWA.

```mermaid
graph TB
    %% User Interface Layer
    subgraph UI["🖥️ User Interface Layer"]
        APP[App.tsx]
        PAGES[Pages:<br/>Login, Home, Vehicles,<br/>StartTrip, EndTrip, etc.]
        COMP[Components:<br/>OfflineStatus, DepartmentSwitcher,<br/>QuickStats, RecentTrips]
    end

    %% Context & State Management
    subgraph CTX["📋 Context & State Management"]
        TRIP_CTX[TripContext]
        NOTIF_CTX[NotificationContext]
        QUERY[React Query<br/>Client]
    end

    %% Main API Router
    subgraph ROUTER["🔀 API Routing Layer"]
        API_ROUTER[ApiRouter.ts<br/>Main Interface]
        NETWORK_MGR[NetworkManager.ts<br/>Network Detection]
    end

    %% Dual API Systems
    subgraph ONLINE["🌐 Online System"]
        ONLINE_API[OnlineApi.ts<br/>Direct HTTP Calls]
        BACKEND_API[Backend API<br/>Express.js]
    end

    subgraph OFFLINE["📴 Offline System"]
        OFFLINE_API[OfflineOnlyApi.ts<br/>IndexedDB Operations]
        OFFLINE_STORAGE[OfflineStorage.ts<br/>IndexedDB Manager]
    end

    %% Storage & Sync
    subgraph STORAGE["💾 Storage & Sync Layer"]
        INDEXEDDB[IndexedDB<br/>Local Storage]
        BG_SYNC[BackgroundSync.ts<br/>Sync Manager]
        SYNC_QUEUE[Sync Queue<br/>Offline Operations]
    end

    %% Vehicle Management
    subgraph VEHICLE["🚗 Vehicle Management"]
        VEHICLE_STATUS[VehicleStatusManager.ts<br/>Status Updates]
        VEHICLE_SYNC[VehicleSyncManager.ts<br/>Sync Operations]
    end

    %% Notifications & PWA
    subgraph PWA["📱 PWA & Notifications"]
        NOTIF_MGR[NotificationManager.ts<br/>Push Notifications]
        SERVICE_WORKER[Service Worker<br/>sw.js]
        PWA_MANIFEST[PWA Manifest<br/>Offline Capabilities]
    end

    %% Backend Database
    subgraph BACKEND["🗄️ Backend Infrastructure"]
        DB[(PostgreSQL<br/>Database)]
        CACHE_SVC[Cache Service<br/>Node Cache]
        PUSH_SVC[Push Notification<br/>Service]
    end

    %% User Interactions
    APP --> PAGES
    APP --> COMP
    PAGES --> CTX
    COMP --> CTX

    %% Context to API Router
    TRIP_CTX --> API_ROUTER
    NOTIF_CTX --> API_ROUTER
    QUERY --> API_ROUTER

    %% API Router Decision Making
    API_ROUTER --> NETWORK_MGR
    NETWORK_MGR -.->|Online| ONLINE_API
    NETWORK_MGR -.->|Offline| OFFLINE_API

    %% Online Flow
    ONLINE_API --> BACKEND_API
    BACKEND_API --> DB
    BACKEND_API --> CACHE_SVC
    BACKEND_API --> PUSH_SVC

    %% Offline Flow
    OFFLINE_API --> OFFLINE_STORAGE
    OFFLINE_STORAGE --> INDEXEDDB
    OFFLINE_API --> SYNC_QUEUE

    %% Background Sync
    BG_SYNC --> SYNC_QUEUE
    BG_SYNC --> ONLINE_API
    NETWORK_MGR -.->|Connection<br/>Restored| BG_SYNC

    %% Vehicle Management
    VEHICLE_STATUS --> API_ROUTER
    VEHICLE_SYNC --> API_ROUTER
    PAGES -.-> VEHICLE_STATUS

    %% Notifications & PWA
    NOTIF_MGR --> SERVICE_WORKER
    SERVICE_WORKER --> PWA_MANIFEST
    PUSH_SVC -.->|Push Events| SERVICE_WORKER
    SERVICE_WORKER -.->|Cache Management| STORAGE

    %% Cache Flow
    ONLINE_API -.->|Cache Success| OFFLINE_STORAGE
    INDEXEDDB -.->|TTL Expired| BG_SYNC

    %% Styling
    classDef uiClass fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef apiClass fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef storageClass fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef pwaCclass fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef backendClass fill:#fce4ec,stroke:#c2185b,stroke-width:2px

    class APP,PAGES,COMP,CTX uiClass
    class API_ROUTER,NETWORK_MGR,ONLINE_API,OFFLINE_API apiClass
    class INDEXEDDB,OFFLINE_STORAGE,BG_SYNC,SYNC_QUEUE,STORAGE storageClass
    class NOTIF_MGR,SERVICE_WORKER,PWA_MANIFEST,PWA pwaCclass
    class DB,CACHE_SVC,PUSH_SVC,BACKEND,BACKEND_API backendClass
```

## 🔄 Communication Flow

### 1. **User Interface Layer**
- **React Components** handle user interactions
- **Pages** manage routing and page-specific logic  
- **Components** provide reusable UI elements

### 2. **Context & State Management**
- **TripContext** manages trip state across components
- **NotificationContext** handles notification logic
- **React Query** provides server state management

### 3. **API Routing Layer**
- **ApiRouter** is the single interface for all API calls
- **NetworkManager** detects online/offline status
- Automatically routes to appropriate API system

### 4. **Dual API Systems**
- **Online System**: Direct HTTP calls to backend
- **Offline System**: Pure IndexedDB operations
- Complete independence between systems

### 5. **Storage & Sync**
- **IndexedDB** stores all offline data
- **BackgroundSync** handles sync when connection returns
- **Sync Queue** manages failed operations

### 6. **Vehicle Management**
- **VehicleStatusManager** handles real-time status updates
- **VehicleSyncManager** manages vehicle data synchronization

### 7. **PWA & Notifications**
- **Service Worker** enables offline functionality
- **NotificationManager** handles push notifications
- **PWA Manifest** provides app-like experience

### 8. **Backend Infrastructure**
- **PostgreSQL** database for persistent storage
- **Cache Service** improves performance with node-cache
- **Push Service** handles notification delivery 