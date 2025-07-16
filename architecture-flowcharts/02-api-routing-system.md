# 🔀 API Routing System

This flowchart shows how the ApiRouter automatically switches between online and offline systems based on network status.

```mermaid
flowchart TD
    START[Component Needs Data] --> API_CALL[Call ApiRouter Method]
    
    API_CALL --> NETWORK_CHECK{NetworkManager<br/>Check Status}
    
    NETWORK_CHECK -->|Online| ONLINE_PATH[Online API Path]
    NETWORK_CHECK -->|Offline| OFFLINE_PATH[Offline API Path]
    
    %% Online Path
    ONLINE_PATH --> ONLINE_API[OnlineApi.ts<br/>HTTP Request]
    ONLINE_API --> BACKEND[Backend API Call]
    
    BACKEND --> SUCCESS{API Call<br/>Successful?}
    SUCCESS -->|Yes| CACHE_DECISION{Cache Online<br/>Data Enabled?}
    SUCCESS -->|No| ERROR_HANDLE[Throw Error<br/>No Fallback]
    
    CACHE_DECISION -->|Yes| CACHE_DATA[Cache Response<br/>to IndexedDB]
    CACHE_DECISION -->|No| RETURN_ONLINE[Return Online Data]
    
    CACHE_DATA --> RETURN_ONLINE
    
    %% Offline Path
    OFFLINE_PATH --> OFFLINE_API[OfflineOnlyApi.ts<br/>IndexedDB Query]
    OFFLINE_API --> INDEXEDDB_CHECK{Data Exists<br/>in IndexedDB?}
    
    INDEXEDDB_CHECK -->|Yes| TTL_CHECK{Data Still<br/>Valid TTL?}
    INDEXEDDB_CHECK -->|No| RETURN_EMPTY[Return Empty<br/>or Default Data]
    
    TTL_CHECK -->|Yes| RETURN_CACHED[Return Cached Data<br/>fromCache: true]
    TTL_CHECK -->|No| EXPIRED_DATA[Data Expired<br/>Return Stale or Empty]
    
    %% Sync Queue for Offline Operations
    OFFLINE_API --> MUTATION_CHECK{Is Write<br/>Operation?}
    MUTATION_CHECK -->|Yes| ADD_SYNC[Add to Sync Queue<br/>for Later Processing]
    MUTATION_CHECK -->|No| INDEXEDDB_CHECK
    
    ADD_SYNC --> RETURN_SUCCESS[Return Success<br/>Queued for Sync]
    
    %% Network Status Change
    NETWORK_STATUS[Network Status Change] --> SYNC_TRIGGER{Connection<br/>Restored?}
    SYNC_TRIGGER -->|Yes| BG_SYNC[BackgroundSync.ts<br/>Process Queue]
    SYNC_TRIGGER -->|No| CONTINUE[Continue Current Mode]
    
    BG_SYNC --> PROCESS_QUEUE[Process Sync Queue<br/>with Online API]
    PROCESS_QUEUE --> RECONCILE[Reconcile Local<br/>and Remote Data]
    
    %% Return Points
    RETURN_ONLINE --> COMPONENT[Component Receives Data]
    RETURN_CACHED --> COMPONENT
    RETURN_EMPTY --> COMPONENT
    RETURN_SUCCESS --> COMPONENT
    ERROR_HANDLE --> COMPONENT
    EXPIRED_DATA --> COMPONENT
    
    %% Vehicle Specific Flow
    subgraph VEHICLE_FLOW["🚗 Vehicle Status Management"]
        VEHICLE_REQ[Vehicle Request] --> VEHICLE_CHECK{Vehicle Status<br/>Cache Fresh?}
        VEHICLE_CHECK -->|No| QUICK_REFRESH[Quick Status Update<br/>VehicleStatusManager]
        VEHICLE_CHECK -->|Yes| USE_CACHE[Use Cached Vehicle Data]
        QUICK_REFRESH --> MERGE_STATUS[Merge Status with<br/>Cached Vehicle Data]
    end
    
    %% Styling
    classDef onlineClass fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef offlineClass fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef decisionClass fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef errorClass fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    classDef syncClass fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class ONLINE_PATH,ONLINE_API,BACKEND,CACHE_DATA,RETURN_ONLINE onlineClass
    class OFFLINE_PATH,OFFLINE_API,INDEXEDDB_CHECK,RETURN_CACHED,ADD_SYNC offlineClass
    class NETWORK_CHECK,SUCCESS,CACHE_DECISION,TTL_CHECK,MUTATION_CHECK,SYNC_TRIGGER decisionClass
    class ERROR_HANDLE,EXPIRED_DATA errorClass
    class BG_SYNC,PROCESS_QUEUE,RECONCILE,VEHICLE_FLOW syncClass
```

## 🔄 API Router Flow Details

### 1. **Network Detection**
- **Continuous Monitoring**: NetworkManager runs periodic connectivity tests
- **Multiple Checks**: Tests browser navigator.onLine + actual backend ping
- **Event Driven**: Components subscribe to network status changes

### 2. **Online Mode Operations**
```typescript
// ApiRouter automatically routes to OnlineApi
const result = await apiRouter.getVehicles();
// → OnlineApi.getVehicles() → HTTP call to backend
// → Optional caching for offline use
```

### 3. **Offline Mode Operations**
```typescript
// Same interface, different implementation
const result = await apiRouter.getVehicles();
// → OfflineOnlyApi.getVehicles() → IndexedDB query
// → Write operations queued for sync
```

### 4. **Background Sync Process**
- **Trigger Events**: Network reconnection, app visibility, manual trigger
- **Priority Queue**: High priority (trips) → Medium → Low priority operations
- **Retry Logic**: Exponential backoff for failed sync attempts
- **Conflict Resolution**: Server data takes precedence, local data reconciled

### 5. **Cache Management**
- **TTL System**: Different cache lifetimes per data type
- **Smart Invalidation**: Related cache cleared on data changes
- **Storage Monitoring**: Track IndexedDB usage and cleanup expired data

### 6. **Vehicle Status Optimization**
- **Separate TTL**: Vehicle status cached for 5 minutes vs 30 minutes for data
- **Quick Updates**: Lightweight status-only refreshes when needed
- **Batch Operations**: Multiple vehicle status updates in single call

## 🎯 Key Benefits

1. **🔄 Seamless Switching**: Components use same interface regardless of network status
2. **⚡ Performance**: Cached data serves instantly, online data cached for offline use
3. **🛡️ Independence**: Offline system failures don't impact online functionality
4. **🔄 Auto-Sync**: Offline operations automatically sync when connection returns
5. **📊 Transparency**: Components know if data is from cache or network 