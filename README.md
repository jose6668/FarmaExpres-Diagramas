# FarmaExpres-Diagramas

## [Diagrama de BPMN](./FarmaExpres_BPMN_MVP_v1_0.pdf)

## Diagramas de clases 

```mermaid
classDiagram




class Usuario {
    +int idUsuario
    +String nombre
    +String correo
    +String passwordHash
    +EstadoUsuario estado
    +autenticar()
    +cambiarPassword()
    +bloquearUsuario()
}



class Rol {
    +int idRol
    +String nombre
    +String descripcion
}

class Bitacora {
    +int idBitacora
    +String accion
    +date fechaHora
}

class Medicamento {
    +int idMedicamento
    +String codigo
    +String nombre
    +decimal precioUnitario
    +Date fechaVencimiento
    +int stock
    +EstadoMedicamento estado
    +actualizarDatos()
    +desactivar()
    +verificarVencimiento()
    +aumentarStock(cantidad)
    +disminuirStock(cantidad)
}

class Movimiento {
    +int idMovimiento
    +TipoMovimiento tipo
    +int cantidad
    +int stockAntes
    +int stockDEspues
    +DateTime fechaHora
    +registrar()
}

class AlertaVencimiento {
    +int idAlerta
    +DateTime fechaGeneracion
    +Date fechaObjetivo
    +int diasAnticipacion
    +EstadoAlerta estado
    +generar()
    +bloquearMedicamento()
}


class Reporte {
    <<abstract>>
    +int idReporte
    +DateTime fechaGeneracion
    +generar()
}


class ReporteInventarioActual
class ReporteProductosAgotados
class ReporteHistorialMovimientos

%% ENUMERACIONES

class EstadoUsuario {
    <<enumeration>>
    Activo
    Inactivo
    Bloqueado
}


class EstadoMedicamento {
    <<enumeration>>
    Activo
    Bloqueado
    Vencido
}

class TipoMovimiento {
    <<enumeration>>
    Entrada
    Salida
    Actualizado
    Eliminado
}

class EstadoAlerta {
    <<enumeration>>
    Activa
    Atendida
}

%% RELACIONES

Usuario "*" --> "1" Rol
Usuario "1" --> "*" Movimiento
Medicamento "1" --> "*" Movimiento
Medicamento "1" --> "*" AlertaVencimiento
Usuario "1" --> "*" Bitacora

Reporte <|-- ReporteInventarioActual
Reporte <|-- ReporteProductosAgotados
Reporte <|-- ReporteHistorialMovimientos

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
DB[(PostgreSQL Unico)]
end

Cliente --> GW
GW --> AUTH
GW --> INV

AUTH --> DB
INV --> DB

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
