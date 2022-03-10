# Architecture

## MVVM
```mermaid
flowchart TD
    subgraph DC[DependencyContainer]
      direction TB
      FileManager
      ErrorTracker
      subgraph Singletons
        direction TB
        Logger
        FFP[<a href='https://github.com/escakot/public-images/blob/master/README.md'>FeatureFlagProvider</a>]
      end
      DCDB[Database]
      DCNW[Networking]
      PM[ProvisioningManager]
      PM -- Inject --> DCNW
      PM -- Inject --> FFP
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
    DC -- Injected by Router -----------> M
```


## Coordinator / Router

### Navigation using Coordinators and Routers
The below diagram is an example of how Navigation should be handled. By using this design pattern, we can navigate to any part of the Application through Universal/Deep Links.


```mermaid
flowchart LR
    DC[DependencyContainer] -- Provides Injection --> AP
    subgraph AC[AppCoordinator]
        AP[AppRouter]
        AP --- ACC[Add/Remove ChildCoordinator]
        ACC -.- FC
        ACC -.- FC2
        AP --- PRVC[Provide RootVC]
        PRVC --> FCR
        PRVC --> FC2R

        subgraph FC[FeatureCoordinator 1]
            FCR[Router]
            subgraph FCVC[ListViewController]
                subgraph FCVM[ViewModel]
                    FCRP[RouterProtocol]
                end
                FCUIA1[UIAction] -- Show Detail --> FCRP
            end
            subgraph FCVC2[DetailViewController]
                subgraph FCVM2[ViewModel]
                    FCRP2[RouterProtocol]
                end
                FCUIA2[UIAction] -- Dismiss --> FCRP2
            end
            FCR --- FCPD[Push Detail]
            FCPD --- FCRP
            FCR --- FCDD[Dismiss Detail]
            FCDD --- FCRP2
            FCR -. Push/Dismiss DetailVC -...-> FCVC2
        end
        subgraph FC2[FeatureCoordinator 2]
            FC2R[Router]
            subgraph FC2VC[ListViewController]
                subgraph FC2VM[ViewModel]
                    FC2RP[RouterProtocol]
                end
                FC2UIA1[UIAction] -- Show Detail --> FC2RP
            end
            subgraph FC2VC2[DetailViewController]
                subgraph FC2VM2[ViewModel]
                    FC2RP2[RouterProtocol]
                end
                FC2UIA2[UIAction] -- Dismiss --> FC2RP2
            end
            FC2R --- FC2PD[Push Detail]
            FC2PD --- FC2RP
            FC2R --- FC2DD[Dismiss Detail]
            FC2DD --- FC2RP2
            FC2R -. Push/Dismiss DetailVC -...-> FC2VC2
        end
        FCR -.- RDC[Request to Dismiss<br>Coordinator] 
        FC2R -.- RDC
        RDC -.-> AP
    end
```
