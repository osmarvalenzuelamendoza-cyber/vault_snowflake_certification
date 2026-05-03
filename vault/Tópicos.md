Micropartions
    Inmutables
    Comprimidos columnarmente
    Clustering depth <> Partition pruning (filtra métricas precalculadas)
    Data recuperation: Time travel + Fail safe
    Metadata only operations

Clustering Keys
	Reordenamiento físico micropartions
	Orden columnas (importa cardinalidad)
    Depende clustering deep
	No clustering manual (activas o suspendes)
    Genera new micropartitions
    Trade off: prunnign vs costo adicional
    Alternativa: Reingesta ordenada
	Ayudate de Tablas diagnosticos

Tablas
	Permanent, Transient, Temporary, Hybrid y External
	Permanent
        Time travel (90d)
        Source dinamico
    Transient
        Source fijo
        Menos costos
        Schemas & Database Transient
    Temporary
        Sesion
    External
        Solo lectura (no partition pruning) (lectura completa)
        No cache
        Costo solo computo
        Particionamiento del path
        Sincronizacion manual o automatica
        Near real time: Auto refresh = True + Materialized Views
        Uso: datalake sin ETL
    Hybrid
        HTAP: OLTP + OLAP
        Doble store
        1 DML > doble update (row y column) > + costo > no usar en tablas analiticas
        PK obligatorio + indice secundario (copia fisica row)(+ costo)
        Row locking: effective dml operations
        X Time travel - X Fail Safe
    
		