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