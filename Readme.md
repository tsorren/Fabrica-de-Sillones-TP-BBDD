# Fábrica de Sillones – OLAP & Relational Database Architecture

Migración de ~522.000 filas de un conjunto de datos desnormalizado hacia un modelo transaccional de 21 tablas en 3FN sobre SQL Server, y posterior diseño de un esquema en estrella (OLAP) con 8 dimensiones, 5 tablas de hechos y 10 vistas de KPI de negocio, con procesos ETL en T-SQL.

## 📑 Índice
- [El Problema y la Motivación](#-el-problema-y-la-motivación)
- [Tech Stack y Arquitectura](#-tech-stack-y-arquitectura)
- [Features Clave](#-features-clave)
- [Aprendizajes Clave y Roadmap](#-aprendizajes-clave-y-roadmap)
- [Visuales y Diagramas de Impacto](#-visuales-y-diagramas-de-impacto)
- [Instalación y Ejecución Local](#️-instalación-y-ejecución-local)

## 🧠 El Problema y la Motivación

El sistema de datos original de la fábrica presentaba serias anomalías de inserción, actualización y borrado al almacenar toda la información operativa en estructuras planas y desnormalizadas. Además de la pérdida de integridad referencial, esto generaba cuellos de botella críticos a la hora de procesar métricas de facturación sobre cientos de miles de registros.

El objetivo de ingeniería de este proyecto fue doble:
1. Diseñar un sistema **OLTP transaccional** robusto capaz de mantener la consistencia operativa en tiempo real con alto volumen de relaciones (ej: la tabla `MaterialPorSillon` con 216.498 registros).
2. Desarrollar un **modelo dimensional OLAP** (Star Schema) independiente, optimizado para resolver consultas analíticas pesadas (10 vistas analíticas críticas) sin degradar el rendimiento del entorno transaccional.

## 🏗️ Tech Stack y Arquitectura

*   **SQL Server / T-SQL:** Motor de base de datos principal; los procedimientos almacenados y los triggers encapsulan la creación del esquema, la migración y la actualización de totales.
*   **Arquitectura OLTP:** Diseño entidad-relación (ER) normalizado hasta la Tercera Forma Normal (3FN). Resuelve redundancias e inconsistencias mediante restricciones y datos depurados en la migración.
*   **Arquitectura OLAP:** Implementación de un modelo dimensional en estrella con 5 tablas de hechos (ventas, compras, pedidos, envíos y ventas por modelo) y 8 dimensiones (tiempo, cliente, ubicación, sucursal, modelo de sillón, tipo de material, estado del pedido y turno de venta), sobre las que se calculan 10 vistas de KPI.

## ⚡ Features Clave

*   **Automatización de Despliegues (ETL):** Todo el proceso de creación de tablas (DDL) y la migración de los datos masivos fue encapsulado en procedimientos almacenados robustos, garantizando un despliegue idempotente y modular.
*   **Mantenimiento Dinámico de Estados:** Implementación de Triggers para la actualización transaccional en cascada de montos totales (`Pedido_Total`, `Compra_Total`, `Factura_Total`), eliminando la necesidad de recálculos al vuelo en cada lectura.
*   **Teardown Automatizado:** Uso avanzado de SQL Dinámico para barrer completamente esquemas y dependencias en entornos efímeros sin arrojar errores de llaves foráneas, facilitando la recreación de la base de datos de manera limpia.

## 📈 Aprendizajes Clave y Roadmap

*   **Optimización de Consultas Analíticas:** Al estructurar la vista de *Rendimiento de Modelos* en el Data Warehouse, se utilizó la función de ventana `ROW_NUMBER() OVER(PARTITION BY...)` encapsulada en un subselect. Esto resultó ser el enfoque más performante para obtener rankings seccionados (Top 3) frente a múltiples anidaciones costosas.
*   **Data Cleansing a Escala:** Migrar desde un dataset legado subrayó la importancia de la calidad de los datos a gran escala; se resolvieron inconsistencias graves como DNI duplicados y registros temporales futuros, blindando el nuevo esquema OLTP con restricciones (CHECKs) y validaciones rigurosas.

## 🖼️ Visuales y Documentación de Impacto

<div align="center">
  <h4>Modelo Transaccional (OLTP)</h4>
  <img src="DER.jpg" alt="Diagrama Entidad-Relación OLTP" width="80%">
  <br><br>
  <h4>Modelo Dimensional (OLAP / Business Intelligence)</h4>
  <img src="DER_BI.jpg" alt="Diagrama Entidad-Relación OLAP" width="80%">
</div>

### 📄 Documentación Anexa
* [**Estrategia de Modelado de Datos (PDF)**](./Fabrica%20de%20Sillones%20-%20Estrategia.pdf): Documento original que detalla el relevamiento de requerimientos, la justificación del pasaje de entidades a tablas, y las reglas de negocio implementadas en el diseño arquitectónico final.

## ⚙️ Instalación y Ejecución Local

Las rutinas de creación y migración están pensadas para ser ejecutadas en un entorno de SQL Server.

```bash
# 1. Clonar el repositorio
git clone https://github.com/tsorren/Fabrica-de-Sillones-TP-BBDD.git
cd Fabrica-de-Sillones-TP-BBDD

# 2. Abrir SQL Server Management Studio (SSMS) o Azure Data Studio

# 3. Ejecutar los scripts en la carpeta 'data/':
# - Primero ejecutar `script_creacion_inicial.sql` (crea el OLTP y migra datos)
# - Luego ejecutar `script_creacion_BI.sql` (crea el OLAP y genera las vistas)

# 4. Para limpiar un despliegue anterior:
# Ejecutar el script `script_rollback.sql`
```
