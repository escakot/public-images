# Feature Flag Provider

## Product Requirements

### Problems:
1. The latest iOS app update is crashing in production. Every user who is up-to-date is unable to use the iOS app. However, releasing updates to the AppStore requires a review process that can take up-to 48 hours.
2. A team of Developers are working on a new Feature. They branch off `master` and complete this Feature after 1 month. When trying to merge the Feature into `master`, there's a ton of merge conflicts. After resolving them and merging, the app crashes and nobody knows why. 

### Solution:
We use Feature Flags(aka Remote Configs) to turn On/Off sections of code for all users. We can disable crashing code the moment we find out. Developers can incrementally work on new Features behind a FF to avoid merge hells.

## Architectural Plan

```mermaid
flowchart LR;
    LDW[LaunchDarkly Website]
    POS[TouchBistro POS]
    LDC[LaunchDarkly CDN]
    BA[Backend APIs]
    UW[User Websites]
    
    LDC <-- FF Sync --> POS
    LDC <-- FF Sync --> BA
    LDC <-- FF Sync ---> UW
   
    POS <-- Communication ---> BA
    
    LDW -- Configures --> LDC
    
    UW <-- Communication ---> BA
```

## Application Modules
```mermaid
flowchart TB;
    subgraph FeatureFlagProvider
        direction LR
        IR[isReachable]
        subgraph LaunchDarklySDK
            LDC[LDClient]
        end
        FBC[FallbackClient]
        IR -- Online --> LDC
        IR -- Offline --> FBC
        FBC -- Online --> LDC
    end
    FeatureFlagProvider -- Provide Flag --> AP[AppFeature Package]
    FeatureFlagProvider -- Provide Flag --> Database
    FeatureFlagProvider -- Provide Flag --> Networking
    FeatureFlagProvider -- Provide Flag --> ViewModel
    FeatureFlagProvider -- Provide Flag --> View/ViewControllers
```

## Transactions and User Flows
```mermaid
sequenceDiagram
    actor Customer
    participant POS
    participant LaunchDarkly
    actor Developer
    actor Customer Support
    
    Customer->>POS: Launch
    POS->>LaunchDarkly: Initial Fetch Flags
    LaunchDarkly->>POS: Return Flags
    POS->>Customer: App Finish Launching
    loop Daily Restaurant Operations
        Customer->>POS: Uses 
        POS->>Customer:  
        loop Polling every 5 minutes
            POS->>LaunchDarkly: Fetch Flags
            LaunchDarkly->>POS: Return Flags
        end
    end
    Customer->>POS: Uses New Feature A
    POS->>Developer: App Crashes and Notifies
    Developer->>LaunchDarkly: Turns Off FeatureFlag A
    loop Polling every 5 minutes
        POS->>LaunchDarkly: Fetch Flags
        LaunchDarkly->>POS: Return Flags
    end
    POS->>Customer: Feature A Unavailable
    Customer->>POS: Uses App without Crashing
    Customer->>POS: Uses Updated Feature B
    POS->>Customer: Not Working as Expected
    Customer->>Customer Support: Calls Customer Support
    Customer Support->>Developer: Contact Developer for fix
    Developer->>LaunchDarkly: Turn Off FeatureFlag B
    loop Polling every 5 minutes
        POS->>LaunchDarkly: Fetch Flags
        LaunchDarkly->>POS: Return Flags
    end
    POS->>Customer: Use Previous Feature B Implementation 
    Customer->>POS: Uses App Feature B that works again 
```

## In-depth Logic Flow (Experimental)

```mermaid
flowchart LR;
    A[App Launch] --> FF;
    SP[Splash Screen];
    subgraph FeatureFlagProvider;
        direction LR;
        FF[Fetch Flags];
        ID[VenueXRefID<br/>Changes];
        FF -- 2s Timeout / No Internet --> FallbackClient;
        FF -- Resolved ----> LaunchDarklyClient;
        subgraph FallbackClient
            direction LR
            UDS[UserDefaults<br/>Storage];
        end
        FallbackClient -. Retry / Internet Available .-> FF;
        subgraph LaunchDarklyClient
            direction LR
            UF[Update Flags];
            SU[Set User];
            SU --> UF;
        end
        LaunchDarklyClient -. Replaces -.-> FallbackClient
        ID ---> SU
        UF -. Update .-> UDS;
    end
    FallbackClient --> SP;
    LaunchDarklyClient --> SP;
```
