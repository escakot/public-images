# Architecture

```mermaid
flowchart TD
    subgraph DC[DependencyContainer]
      direction TB
      FileManager
      ErrorTracker
      subgraph Singletons
        direction TB
        Logger
        FFP[<a href='./README.md'>FeatureFlagProvider</a>]
      end
      DCDB[Database]
      DCNW[Networking]
      PM[ProvisioningManager]
      PM -- Inject --> DCNW
    end
    subgraph App Feature
      direction LR
      subgraph VM[ViewModel]
        direction LR
        Data
        FD([Fetch Data?])
        FD --> LI[Logged In?]
        LI --> VMY([Yes])
        LI --> VMN([No])
        VMY -- Request Data ----> M
        subgraph M[Model]
          direction LR
          NW[Networking]
          DB[Database]
          FR[Fetch Request] --> DB
          DB -- If No Data --> NW
          NW -- Store Fetched Data --> DB
          DB -- Provides --> Data[In Memory Data]
          NW -- Request Data --> MD[Main Device]
          MD -- Response Data --> NW
        end
      end
      VMN -- Send Error --> VC
      Data -- Notifies --> VC
      subgraph V[View]
        direction LR
        VC[ViewController]
        VC -- Updates --> View
      end
      V -- Sends UIEvents ----> FD
    end
    DC -- Inject ----> M
```
