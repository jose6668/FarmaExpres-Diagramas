# FarmaExpres-Diagramas

##Diagramas de clases 

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

Reporte <|-- ReporteInventarioActual
Reporte <|-- ReporteProductosAgotados
Reporte <|-- ReporteHistorialMovimientos

```

