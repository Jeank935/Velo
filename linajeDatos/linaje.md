# Informe General de Linaje de Datos (Data Lineage)
**Sistema de Analítica Geográfica Comercial y Operativa (Lojagas)**

## 1. Objetivo del Linaje de Datos
El presente informe detalla el ciclo de vida, la trazabilidad y la transformación de la información dentro de la arquitectura del sistema Lojagas. El objetivo principal de este flujo es garantizar la ingesta, limpieza, procesamiento y visualización de datos geográficos y comerciales, cumpliendo con la restricción arquitectónica fundamental: **el aislamiento absoluto del *core* transaccional (Odoo ERP) para evitar la degradación de su rendimiento operativo**.

## 2. Descripción del Flujo de Datos (Pipeline Arquitectónico)
El ciclo de vida de los datos se estructura en cinco fases lógicas secuenciales, operando bajo un modelo de procesamiento tipo *Pipeline* (tubería de datos):

### Fase 1: Orígenes de Datos (Data Sources)
La arquitectura consume información de tres fuentes heterogéneas:
*   **Odoo ERP (Core Transaccional):** Actúa como la "Fuente Única de Verdad" (*Single Source of Truth*) para los datos maestros y transaccionales del negocio.
*   **Datos Externos (Competidores):** Información estructurada (archivos plana o APIs de terceros) que provee inteligencia de mercado sobre la presencia de la competencia.
*   **Servicios Geoespaciales Cloud (OSM / Mapbox):** Proveedores externos (SaaS) que suministran las coordenadas de geolocalización y los mapas base (Tiles).

### Fase 2: Ingesta y Calidad (Ingestion & Data Quality)
Para garantizar el desacoplamiento, los datos no se consultan en tiempo real a la base de datos de Odoo.
*   **Worker Asíncrono (Extracción):** Un componente que se comunica mediante protocolos estándar (JSON/RPC) con Odoo para extraer datos de forma incremental y programada (procesamiento por lotes o *micro-batches*).
*   **Limpieza y Deduplicación:** Antes de la persistencia, los datos atraviesan un componente de validación que estandariza los formatos, elimina registros duplicados y gestiona valores nulos, asegurando la integridad referencial (Cumplimiento norma ISO/IEC 25010 en Adecuación Funcional).

### Fase 3: Almacenamiento y Persistencia (Data Storage)
Los datos validados ("Carga Limpia") son almacenados de forma definitiva en un entorno optimizado para consultas matemáticas espaciales:
*   **Base de Datos Analítica (PostGIS):** Un clúster independiente configurado con índices espaciales (GiST).
*   **Mapeo a Entidades de Dominio:** La información relacional se transforma en colecciones de dominio orientadas al negocio: *Clientes/Distribuidores*, *Ventas/Productos* y polígonos de *Zonas Territoriales/Competidores*.

### Fase 4: Capa de Procesamiento (Data Processing)
Las entidades persistidas son consumidas por dos motores de lógica de negocio (Capa de Control) que actúan en paralelo:
*   **Motor GIS (Analizador Espacial):** Consume las entidades y las cruza con las coordenadas de servicios externos (Mapbox/OSM) para procesar cálculos de geolocalización, densidad poblacional, polígonos de influencia e intersecciones de competencia.
*   **Motor KPI (Servicio de Información Comercial):** Agrupa las métricas de ventas y cobertura, calculando indicadores de rendimiento, brechas de mercado y proyecciones de crecimiento.

### Fase 5: Visualización y Consumo (Boundary / Frontend)
Los datos transformados en conocimiento estratégico son servidos a las interfaces de usuario (Frontend), optimizados para el rendimiento visual:
*   **Visor Cartográfico (Mapas Interactivos):** El Motor GIS no envía tablas, sino que renderiza y transmite **Vectores MVT (Mapbox Vector Tiles)**. Esta táctica arquitectónica permite al cliente dibujar miles de puntos, mapas de calor y *clusters* en el navegador en menos de 2 segundos, sin sobrecargar la memoria del dispositivo.
*   **Tablero Analítico (Dashboards):** El Motor KPI sirve métricas sumarizadas en formato JSON ligero, permitiendo la construcción de resúmenes ejecutivos dinámicos bajo políticas de control de acceso basado en roles (RBAC).

## 3. Conclusión Arquitectónica
Este grafo de dependencias y linaje certifica que la solución para Lojagas es **altamente cohesiva y débilmente acoplada**. Odoo queda protegido en la Fase 1, operando puramente como facturador y ERP, mientras que la pesada carga del álgebra espacial (PostGIS, MVT) y de la analítica de negocios (Rankings, KPIs) ocurre en una infraestructura totalmente apartada (Fases 3 y 4), garantizando estabilidad, seguridad y tolerancia a fallos.
