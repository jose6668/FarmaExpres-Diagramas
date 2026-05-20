# Diagrama de clases del microservicio de predicción NoSQL

Este documento explica cómo está organizado el microservicio de predicción de FarmaExpres. El módulo no reemplaza el backend principal: toma datos del inventario relacional, los guarda en MongoDB, limpia los registros y calcula una predicción sencilla de demanda para apoyar la reposición de medicamentos.

## Alcance del diagrama

El diagrama representa las clases y módulos principales del servicio Python con FastAPI:

- `IngestRequest` define desde dónde se cargan los datos: inventario, PostgreSQL o datos generados para pruebas.
- `TrainRequest` define cuántos días se quieren proyectar.
- `PredictionApi` agrupa los endpoints públicos y los endpoints integrados bajo `/api/predictions`.
- `Settings` centraliza variables de ambiente como MongoDB, URL del inventario y seguridad JWT.
- `MongoDatabase` representa el acceso a MongoDB y sus colecciones.
- `AuthContext` representa la información del usuario autenticado que llega desde el token JWT.
- `InventoryClient` consulta el snapshot del `inventory-service`.
- `IngestionService` transforma la información relacional o del inventario en documentos crudos y snapshots de productos.
- `CleaningService` normaliza los registros, elimina duplicados y marca problemas de calidad.
- `PredictionService` calcula la demanda esperada usando promedio móvil de 30 días.
- Las colecciones de MongoDB separan datos crudos, datos limpios, predicciones, métricas y copia de productos.

## Diagrama

```mermaid
classDiagram
direction LR

class IngestRequest {
    +string source
    +int product_count
    +int days
}

class TrainRequest {
    +int horizon_days
}

class PredictionApi {
    +health()
    +seed_test_data(request)
    +ingest(request, context)
    +clean()
    +train(request)
    +recalculate(request)
    +list_predictions(limit)
    +get_prediction(product_id)
    +metrics()
}

class Settings {
    +string mongo_uri
    +string mongo_database
    +string relational_db_url
    +string inventory_service_url
    +int inventory_service_timeout_ms
    +string jwt_secret
    +string environment
    +bool enable_dev_seed_endpoints
    +list cors_origins
}

class AuthContext {
    +string token
    +string user_id
    +string email
    +string name
    +string role
}

class AuthService {
    +get_auth_context(authorization)
    +require_roles(roles)
    +ensure_dev_seed_enabled(context, roles)
}

class MongoDatabase {
    +get_client()
    +get_database()
    +ensure_indexes(db)
    +ping_database()
}

class InventoryClient {
    +fetch_inventory_snapshot(context)
}

class IngestionService {
    +replace_generated_data(db, product_count, days)
    +ingest_from_inventory_snapshot(db, snapshot)
    +ingest_from_postgres(db, relational_db_url)
    -replace_active_training_dataset(db, raw_records, snapshots)
    -replace_products_snapshot(db, snapshots)
    -clear_derived_collections(db)
}

class SampleDataService {
    +generate_synthetic_data(product_count, days, seed)
}

class CleaningService {
    +clean_records(raw_records)
    +normalize_name(value)
    +parse_date(value)
}

class PredictionService {
    +build_predictions(cleaned_records, snapshots, horizon_days)
    -risk_level(stock, minimum_stock, predicted_units, daily_average)
    -calculate_mae(daily_amounts)
}

class raw_data {
    <<Mongo collection>>
    +source
    +product_id
    +movement_type
    +amount
    +movement_date
    +stock
}

class cleaned_data {
    <<Mongo collection>>
    +product_id
    +normalized_product_name
    +movement_type
    +amount
    +movement_date
    +is_valid
    +quality_flags
}

class products_snapshot {
    <<Mongo collection>>
    +product_id
    +product_code
    +product_name
    +category
    +current_stock
    +minimum_stock
}

class predictions {
    <<Mongo collection>>
    +product_id
    +predicted_demand_units
    +moving_average_daily
    +estimated_stockout_days
    +risk_level
    +method
}

class model_metrics {
    <<Mongo collection>>
    +type
    +trained_at
    +valid_records_used
    +average_mae
    +risk_high_count
}

PredictionApi --> IngestRequest : recibe
PredictionApi --> TrainRequest : recibe
PredictionApi --> AuthService : valida roles
PredictionApi --> MongoDatabase : usa
PredictionApi --> IngestionService : ingesta datos
PredictionApi --> CleaningService : limpia datos
PredictionApi --> PredictionService : calcula predicciones

AuthService --> AuthContext : crea
AuthService --> Settings : usa secreto JWT
InventoryClient --> AuthContext : reenvía token
InventoryClient --> Settings : usa URL del inventario
IngestionService --> InventoryClient : recibe snapshot
IngestionService --> SampleDataService : genera datos locales
IngestionService --> raw_data : escribe
IngestionService --> products_snapshot : escribe
IngestionService --> cleaned_data : reinicia
IngestionService --> predictions : reinicia
CleaningService --> raw_data : lee
CleaningService --> cleaned_data : prepara
PredictionService --> cleaned_data : lee salidas válidas
PredictionService --> products_snapshot : lee stock actual
PredictionService --> predictions : escribe resultado
PredictionService --> model_metrics : registra métricas
MongoDatabase --> raw_data : indexa
MongoDatabase --> cleaned_data : indexa
MongoDatabase --> predictions : indexa
MongoDatabase --> model_metrics : indexa
MongoDatabase --> products_snapshot : indexa
```

## Explicación del funcionamiento

El flujo inicia cuando el frontend o el API Gateway llama al endpoint de ingesta. Si la fuente es `inventory`, el microservicio usa el token JWT del usuario y consulta el endpoint de snapshot del `inventory-service`. Ese snapshot contiene productos, lotes y movimientos del inventario relacional. El microservicio no modifica esos datos en PostgreSQL; solo los lee y crea una copia analítica en MongoDB.

La colección `raw_data` almacena los movimientos tal como llegan o como fueron generados para pruebas. La colección `products_snapshot` guarda el estado resumido de cada medicamento, especialmente nombre, código, categoría, stock actual y stock mínimo. Con esa separación, el modelo puede calcular demanda histórica sin depender todo el tiempo de consultas directas al sistema relacional.

Después se ejecuta la limpieza. `CleaningService` convierte fechas a formato estándar, normaliza nombres de medicamentos, elimina duplicados, valida cantidades negativas o faltantes y marca registros incompletos con `quality_flags`. Los registros resultantes se guardan en `cleaned_data`. Los registros inválidos no desaparecen del análisis: quedan identificados para que las métricas indiquen cuántos datos no deberían usarse para calcular demanda.

El entrenamiento lo realiza `PredictionService`. El algoritmo usado es un promedio móvil de 30 días sobre movimientos de tipo `Exit`, porque las salidas de inventario representan demanda histórica. Para cada medicamento se calcula el promedio diario reciente, se multiplica por el horizonte configurado, normalmente 7 días, y se estima si el stock actual alcanza para cubrir esa demanda. Con esa información se clasifica el riesgo como `OUT_OF_STOCK`, `HIGH`, `MEDIUM` o `LOW`.

Finalmente, las predicciones se guardan en `predictions` y las métricas del proceso se guardan en `model_metrics`. El frontend consulta esas colecciones por medio de la API para mostrar demanda esperada, riesgo de agotamiento, prioridad de reposición, cantidad de datos limpios y error medio aproximado.

## Relación con el backend principal

El backend principal conserva la lógica operativa de FarmaExpres: usuarios, autenticación, medicamentos, inventario, movimientos y alertas. El microservicio predictivo queda como una capa analítica independiente. La integración profesional se hace por medio del API Gateway y del `inventory-service`, no escribiendo directamente en las tablas relacionales.

Cuando se ejecuta en Docker, cada ambiente (`dev`, `qa` o `main`) tiene sus propias variables de entorno, base MongoDB y puertos. Esto permite probar el modelo sin mezclar datos entre ambientes.

## Decisiones representadas

- MongoDB se usa para guardar datos flexibles de análisis, no para reemplazar PostgreSQL.
- PostgreSQL sigue siendo la fuente operativa de verdad del inventario.
- La limpieza de datos ocurre antes de entrenar o recalcular predicciones.
- La predicción usa un método simple, verificable y fácil de sustentar: promedio móvil de 30 días.
- El modelo prioriza medicamentos por demanda esperada y riesgo de agotamiento, que son variables útiles para reposición.
- Los datos generados son solo para pruebas y validación local, no para producción.
