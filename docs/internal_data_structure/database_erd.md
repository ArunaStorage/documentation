
This section provides a simplified overview of the ArunaServer database schema through an entity-relationship diagram. 


=== ":diagramsdotnet-diagramsdotnet: draw.io"

    <figure markdown>
      ![Image title](database-erd.light.png#only-light){ loading=lazy }
      ![Image title](database-erd.dark.png#only-dark){ loading=lazy }
      <figcaption>Aruna database entity-relationship diagram. Click for zoom.</figcaption>
    </figure>


=== ":mermaidjs-mermaidjs: Mermaid.js"

    ``` mermaid
    erDiagram

      users ||--o{ objects : "creates / owns"
      users ||--o{ stream_consumers : "creates / owns"
      users ||--o{ persistent_notifications : "receives / acknowledges"
      users ||--o{ hooks : "is owner"
      users ||--o{ workspaces : "is owner"
      users {
        UUID id PK
        TEXT display_name
        VARCHAR(511) email
        VARCHAR(511) external_id
        JSON attributes
        Boolean active
      }

      endpoints {
        UUID id PK
        TEXT name
        JSON host_config
        EndpointVariant endpoint_variant
        UUID documentation_object FK
        Boolean is_public
        EndpointStatus status
      }

      pub_keys ||--|| endpoints : "provides public key"
      pub_keys {
        SMALLSERIAL id PK
        UUID proxy FK
        TEXT pubkey "UNIQUE"
      }

      licenses ||--|| objects : "Metadata license"
      licenses ||--|| objects : "Data license"
      licenses {
        VARCHAR(511) tag PK
        VARCHAR(511) name
        VARCHAR(1023) description
        VARCHAR(511) url
      }

      objects ||--o{ endpoints : "describes"
      objects ||--|{ internal_relations : "is target"
      objects ||--|{ internal_relations : "is origin"
      objects {
        UUID id PK,UK
        Int revision_number
        VARCHAR(511) name
        VARCHAR(1023) description
        Timestamp created_at
        UUID created_by FK
        Int64 content_len
        Int count
        JSON key_values
        ObjectStatus object_status
        DataClass data_class
        ObjectType object_type UK 
        JSON external_relations
        JSON hashes
        Boolean dynamic
        JSON endpoints
        VARCHAR(511) metadata_license FK
        VARCHAR(511) data_license FK
      }

      relation_types ||--o{ internal_relations : "provides relation name"
      relation_types {
        VARCHAR(511) relation_name PK
      }

      internal_relations {
        UUID id PK
        UUID origin_pid FK,UK
        ObjectType origin_type FK
        VARCHAR(511) relation_name FK,UK
        UUID target_pid FK,UK
        ObjectType target_type FK
        VARCHAR(511) target_name
      }

      stream_consumers {
        UUID id PK
        UUID user_id FK
        JSON config
      }

      persistent_notifications {
        UUID id PK
        UUID user_id FK
        PersistentNotificationVariant notification_variant
        TEXT message
        JSON refs
      }

      hooks {
        UUID id PK
        VARCHAR(511) name
        VARCHAR(1023) description
        UUID[] project_ids
        UUID owner FK
        JSON trigger
        TIMESTAMP timeout
        JSON hook
      }

      workspaces {
        UUID id PK
        VARCHAR(511) name
        VARCHAR(1023) description
        UUID owner FK
        VARCHAR(511) prefix
        JSON hook_ids
        JSON endpoint_ids
      }
    ```
