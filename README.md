# FarmaExpres-Diagramas

## [Diagrama de BPMN](./FarmaExpres_BPMN_MVP_v1_0.pdf)

## Diagramas de clases 

```mermaid
classDiagram


class User {
    +int idUser
    +String name
    +String email
    +String passwordHash
    +UserStatus State
    +authenticate()
    +changePassword()
    +BlockUser()
}



class Role {
    +int idRole
    +String name
    +String description
}

class Binnacle {
    +int idBinnacle
    +String action
    +date dateTime
}

class product {
    +int idproduct
    +String code
    +String name
    +decimal Unitprice
    +Date ExpirationDate
    +int stock
    +ProductState State
    +createProduct()
    +updateProduct()
    +removeProduct()
    +listProducts()
    +outofstockproducts()
}

class Motion {
    +int idMotion
    +MovementType Type
    +int amount
    +int stockBefore
    +int stockDesuer
    +DateTime dateTime
    +register()
}

class ExpirationAlert {
    +int idAlert
    +DateTime generationdate
    +Date ExpirationDate
    +int daysAnticipation
    +StateofAlert state
    +trigger()
    +blockproduct()
}


class Report {
    <<abstract>>
    +int idReport
    +DateTime generationdate
    +trigger()
}


%% ENUMERATIONS

class UserStatus {
    <<enumeration>>
    Asset
    Idle
    Blocked
}


class ProductStatus {
    <<enumeration>>
    Asset
    Blocked
    Defeated
}

class MovementType {
    <<enumeration>>
    Entrance
    Exit
    Updated
    Deleted
}


%% RELACIONES

User "*" --> "1" Role
User "1" --> "*" Motion
product "1" --> "*" Motion
product "1" --> "*" ExpirationAlert
User "1" --> "*" Binnacle


```
## Diagrama de casos de usos
```mermaid

flowchart LR

Administrador([Administrador])
Empleado([Empleado])
Sistema([Sistema])

subgraph FarmaExpres

UC1([Gestionar Usuarios])
UC2([Autenticarse])
UC3([Cambiar Contraseña])

UC4([Registrar Medicamento])
UC5([Actualizar Medicamento])


UC7([Consultar Inventario])

UC8([Generar Alertas de Vencimiento])

UC11([Ver Historial de Movimientos])

end

%% Relaciones Administrador
Administrador --> UC1
Administrador --> UC2
Administrador --> UC3
Administrador --> UC4
Administrador --> UC5
Administrador --> UC7
Administrador --> UC11

%% Relaciones Farmacéutico
Empleado --> UC2
Empleado --> UC7



%% Sistema genera alertas automáticamente
Sistema --> UC8
```

## Diagrama de Arquitectura Base

```mermaid
flowchart LR

Cliente([Cliente REST / Postman])

subgraph Gateway
GW[API Gateway]
end

subgraph Servicios
AUTH[Auth Service]
INV[Inventory Service]
end

subgraph BaseDeDatos
DBL[(PostgreSQL login)]
DBI[(PostgreSQL Inventory)]
end

Cliente --> GW
GW --> AUTH
GW --> INV

AUTH --> DBL
INV --> DBI

```

### Diagrama de Contexto

```mermaid
flowchart LR

Security[Contexto Seguridad]
Inventory[Contexto Inventario]
Movement[Contexto Movimientos]
Monitoring[Contexto Alertas y Reportes]

Security --> Inventory
Inventory --> Movement
Inventory --> Monitoring
Movement --> Monitoring

```
