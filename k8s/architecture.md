graph TD
subgraph "User Traffic"
direction LR
User[<font size=5>👤</font><br>User]
end

    subgraph "Kubernetes Cluster (Kind)"
        direction TB

        subgraph "Networking"
            Ingress[<font size=4>🌐</font><br><b>Ingress</b><br><br>easyshop-ingress<br><i>Routes traffic from<br>public-ip-address.nip.io</i>]
        end

        subgraph "EasyShop Application"
            direction LR
            HPA[<font size=4>↔️</font><br><b>HorizontalPodAutoscaler</b><br><br>easyshop-hpa<br><i>Scales pods 2-4<br>based on CPU > 50%</i>]
            EasyShopService[<font size=4>⚙️</font><br><b>Service (NodePort)</b><br><br>easyshop-service<br><i>Port 80 ➔ 3000</i>]

            subgraph "Deployment: easyshop"
                direction TB
                ReplicaSet(ReplicaSet)
                Pod1[<font size=4>📦</font><br>Pod: easyshop-1]
                Pod2[<font size=4>📦</font><br>Pod: easyshop-2]
                ReplicaSet --> Pod1
                ReplicaSet --> Pod2
            end
        end

        subgraph "MongoDB Database"
            direction LR
            VPA[<font size=4>↕️</font><br><b>VerticalPodAutoscaler</b><br><br>easyshop-vpa<br><i>Auto-adjusts<br>MongoDB resources</i>]
            MongoDBService[<font size=4>⚙️</font><br><b>Service (ClusterIP)</b><br><br>mongodb-service<br><i>Port 27017</i>]

            subgraph "StatefulSet: mongodb"
                direction TB
                MongoPod[<font size=4>📦</font><br>Pod: mongodb-0]
            end
        end

        subgraph "Storage"
            direction TB
            PVC[<font size=4>💾</font><br><b>PersistentVolumeClaim</b><br><br>mongodb-pvc<br><i>Requests 5Gi</i>]
            PV[<font size=4>💽</font><br><b>PersistentVolume</b><br><br>mongodb-pv<br><i>8Gi HostPath</i>]
        end

        subgraph "Configuration & Jobs"
            direction TB
            ConfigMap[<font size=4>📄</font><br><b>ConfigMap</b><br><br>easyshop-config]
            Secret[<font size=4>🔑</font><br><b>Secret</b><br><br>easyshop-secrets]
            MigrationJob[<font size=4>🏃</font><br><b>Job</b><br><br>db-migration]
        end

        %% Connections
        User --> Ingress
        Ingress --> EasyShopService
        EasyShopService --> Pod1
        EasyShopService --> Pod2

        HPA -.-> ReplicaSet

        Pod1 --> MongoDBService
        Pod2 --> MongoDBService
        MigrationJob --> MongoDBService

        MongoDBService --> MongoPod

        VPA -.-> MongoPod

        MongoPod -- "mounts /data/db" --> PVC
        PVC -- "binds to" --> PV

        ConfigMap -.-> Pod1
        Secret -.-> Pod1
        ConfigMap -.-> Pod2
        Secret -.-> Pod2
    end

    classDef default fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef pod fill:#e0f7fa,stroke:#00796b,stroke-width:2px;
    classDef storage fill:#fffde7,stroke:#f57f17,stroke-width:2px;
    classDef service fill:#e3f2fd,stroke:#1565c0,stroke-width:2px;
    classDef ingress fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px;
    classDef autoscaler fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef config fill:#fbe9e7,stroke:#d84315,stroke-width:2px;

    class User default
    class Ingress ingress
    class EasyShopService,MongoDBService service
    class Pod1,Pod2,MongoPod pod
    class PVC,PV storage
    class HPA,VPA autoscaler
    class ConfigMap,Secret,MigrationJob config
