# FarmaExpres-Diagramas

## Diagrama BPMN

- [Proceso BPMN del MVP](./FarmaExpres_BPMN_MVP_v1_0.pdf)

## Diagrama de clases

```mermaid
classDiagram

class User {
    +number id
    +string name
    +string email
    +string passwordHash
    +UserStatus status
    +authenticate()
    +changePassword()
    +updateProfile()
}

class Role {
    +number id
    +string code
    +string name
}

class Medicine {
    +number id
    +string code
    +string name
    +string genericName
    +string concentration
    +string dosageForm
    +string presentation
    +number stock
    +number minimumStock
    +number maximumStock
    +number unitPrice
    +boolean active
    +create()
    +update()
    +deactivate()
    +list()
}

class Batch {
    +number id
    +string batchCode
    +date expirationDate
    +number availableStock
    +BatchStatus status
}

class Movement {
    +number id
    +MovementType type
    +number amount
    +string reason
    +datetime dateTime
    +string observation
    +registerEntry()
    +registerExit()
    +list()
}

class Alert {
    +string id
    +AlertType type
    +string status
    +date expirationDate
    +number daysRemaining
    +number batchStock
    +number operationalStock
    +generate()
    +list()
}

class Report {
    <<abstract>>
    +string key
    +string label
    +generate()
    +export()
}

class InventoryReport {
}

class MovementsReport {
}

class ExpiringReport {
}

class LowStockReport {
}

class ByUserReport {
}

class UserStatus {
    <<enumeration>>
    ACTIVE
    BLOCKED
    INACTIVE
}

class BatchStatus {
    <<enumeration>>
    ACTIVE
    EXPIRED
    BLOCKED
}

class MovementType {
    <<enumeration>>
    ENTRANCE
    EXIT
    UPDATED
    DELETED
}

class AlertType {
    <<enumeration>>
    EXPIRED
    EXPIRING_SOON
    LOW_STOCK
    OUT_OF_STOCK
}

User "*" --> "1" Role
User "1" --> "*" Movement
Medicine "1" --> "*" Batch
Medicine "1" --> "*" Movement
Batch "1" --> "*" Movement
Medicine "1" --> "*" Alert
Batch "0..1" --> "*" Alert
Report <|-- InventoryReport
Report <|-- MovementsReport
Report <|-- ExpiringReport
Report <|-- LowStockReport
Report <|-- ByUserReport
```

## Diagrama de casos de uso

```mermaid
flowchart LR

Administrador([Administrador])
Farmaceutico([Farmaceutico])
Auditor([Auditor])
Sistema([Sistema])

subgraph FarmaExpres
UC1([Autenticarse])
UC2([Gestionar usuarios])
UC3([Cambiar contrasena])
UC4([Consultar medicamentos])
UC5([Registrar medicamento])
UC6([Editar medicamento])
UC7([Desactivar medicamento])
UC8([Registrar entrada])
UC9([Registrar salida])
UC10([Consultar movimientos])
UC11([Consultar alertas])
UC12([Generar reportes])
UC13([Generar alertas de vencimiento y stock])
end

Administrador --> UC1
Administrador --> UC2
Administrador --> UC3
Administrador --> UC4
Administrador --> UC5
Administrador --> UC6
Administrador --> UC7
Administrador --> UC10
Administrador --> UC11
Administrador --> UC12

Farmaceutico --> UC1
Farmaceutico --> UC3
Farmaceutico --> UC4
Farmaceutico --> UC8
Farmaceutico --> UC9
Farmaceutico --> UC11

Auditor --> UC1
Auditor --> UC4
Auditor --> UC10
Auditor --> UC12

Sistema --> UC13
```

## Diagrama de arquitectura base

```mermaid
flowchart LR

Cliente([Frontend React / Vite])

subgraph Frontend
APP[Modulos: auth, users, medicines, entries, exits, movements, alerts, reports]
end

subgraph Backend
GW[API Gateway]
AUTH[Auth Service]
INV[Inventory Service]
REP[Reporting Service]
ALR[Alerts Service]
end

subgraph Datos
DBAUTH[(PostgreSQL Auth)]
DBINV[(PostgreSQL Inventory)]
end

Cliente --> APP
APP --> GW
GW --> AUTH
GW --> INV
GW --> REP
GW --> ALR
AUTH --> DBAUTH
INV --> DBINV
REP --> DBINV
ALR --> DBINV
```

## Diagrama de contexto

```mermaid
flowchart LR

Security[Contexto de Seguridad]
Users[Contexto de Usuarios]
Medicines[Contexto de Medicamentos]
Inventory[Contexto de Inventario]
Movements[Contexto de Movimientos]
Alerts[Contexto de Alertas]
Reports[Contexto de Reportes]

Security --> Users
Security --> Medicines
Users --> Movements
Medicines --> Inventory
Inventory --> Movements
Inventory --> Alerts
Movements --> Reports
Alerts --> Reports
```

## Alineacion con el frontend actual

Este repositorio de diagramas queda alineado con los modulos funcionales actualmente implementados en el frontend:

- `auth`
- `users`
- `medicines`
- `entries`
- `exits`
- `movements`
- `alerts`
- `reports`

Notas:

- `dashboard` y `audit` aparecen en la referencia visual, pero no forman parte de las rutas funcionales actuales del frontend.
- Los reportes actualmente contemplados en la interfaz son: inventario actual, movimientos, proximos a vencer, bajo stock y por usuario.
- Las alertas actuales contemplan: vencidos, proximos a vencer, bajo stock y agotados.
