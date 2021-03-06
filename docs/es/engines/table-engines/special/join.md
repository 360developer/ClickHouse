---
machine_translated: true
machine_translated_rev: 3e185d24c9fe772c7cf03d5475247fb829a21dfa
toc_priority: 40
toc_title: Unir
---

# Unir {#join}

Estructura de datos preparada para usar en [JOIN](../../../sql-reference/statements/select.md#select-join) operación.

## Creación De Una Tabla {#creating-a-table}

``` sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
) ENGINE = Join(join_strictness, join_type, k1[, k2, ...])
```

Vea la descripción detallada del [CREATE TABLE](../../../sql-reference/statements/create.md#create-table-query) consulta.

**Parámetros del motor**

-   `join_strictness` – [ÚNETE a la rigurosidad](../../../sql-reference/statements/select.md#select-join-strictness).
-   `join_type` – [Tipo de unión](../../../sql-reference/statements/select.md#select-join-types).
-   `k1[, k2, ...]` – Key columns from the `USING` cláusula que el `JOIN` operación se hace con.

Entrar `join_strictness` y `join_type` parámetros sin comillas, por ejemplo, `Join(ANY, LEFT, col1)`. Deben coincidir con el `JOIN` operación para la que se utilizará la tabla. Si los parámetros no coinciden, ClickHouse no lanza una excepción y puede devolver datos incorrectos.

## Uso De La Tabla {#table-usage}

### Ejemplo {#example}

Creación de la tabla del lado izquierdo:

``` sql
CREATE TABLE id_val(`id` UInt32, `val` UInt32) ENGINE = TinyLog
```

``` sql
INSERT INTO id_val VALUES (1,11)(2,12)(3,13)
```

Creando el lado derecho `Join` tabla:

``` sql
CREATE TABLE id_val_join(`id` UInt32, `val` UInt8) ENGINE = Join(ANY, LEFT, id)
```

``` sql
INSERT INTO id_val_join VALUES (1,21)(1,22)(3,23)
```

Unirse a las tablas:

``` sql
SELECT * FROM id_val ANY LEFT JOIN id_val_join USING (id) SETTINGS join_use_nulls = 1
```

``` text
┌─id─┬─val─┬─id_val_join.val─┐
│  1 │  11 │              21 │
│  2 │  12 │            ᴺᵁᴸᴸ │
│  3 │  13 │              23 │
└────┴─────┴─────────────────┘
```

Como alternativa, puede recuperar datos del `Join` tabla, especificando el valor de la clave de unión:

``` sql
SELECT joinGet('id_val_join', 'val', toUInt32(1))
```

``` text
┌─joinGet('id_val_join', 'val', toUInt32(1))─┐
│                                         21 │
└────────────────────────────────────────────┘
```

### Selección e inserción De Datos {#selecting-and-inserting-data}

Usted puede utilizar `INSERT` consultas para agregar datos al `Join`-mesas de motor. Si la tabla se creó con el `ANY` estricta, se ignoran los datos de las claves duplicadas. Con el `ALL` estricta, se agregan todas las filas.

No se puede realizar una `SELECT` consulta directamente desde la tabla. En su lugar, use uno de los siguientes métodos:

-   Coloque la mesa hacia el lado derecho en un `JOIN` clausula.
-   Llame al [joinGet](../../../sql-reference/functions/other-functions.md#joinget) función, que le permite extraer datos de la tabla de la misma manera que de un diccionario.

### Limitaciones y Ajustes {#join-limitations-and-settings}

Al crear una tabla, se aplican los siguientes valores:

-   [Sistema abierto.](../../../operations/settings/settings.md#join_use_nulls)
-   [Método de codificación de datos:](../../../operations/settings/query-complexity.md#settings-max_rows_in_join)
-   [Método de codificación de datos:](../../../operations/settings/query-complexity.md#settings-max_bytes_in_join)
-   [join\_overflow\_mode](../../../operations/settings/query-complexity.md#settings-join_overflow_mode)
-   [join\_any\_take\_last\_row](../../../operations/settings/settings.md#settings-join_any_take_last_row)

El `Join`-las tablas del motor no se pueden usar en `GLOBAL JOIN` operación.

El `Join`-motor permite el uso [Sistema abierto.](../../../operations/settings/settings.md#join_use_nulls) ajuste en el `CREATE TABLE` instrucción. Y [SELECT](../../../sql-reference/statements/select.md) consulta permite el uso `join_use_nulls` demasiado. Si tienes diferentes `join_use_nulls` configuración, puede obtener un error al unirse a la tabla. Depende del tipo de JOIN. Cuando se utiliza [joinGet](../../../sql-reference/functions/other-functions.md#joinget) función, usted tiene que utilizar el mismo `join_use_nulls` ajuste en `CRATE TABLE` y `SELECT` instrucción.

## Almacenamiento De Datos {#data-storage}

`Join` datos de la tabla siempre se encuentra en la memoria RAM. Al insertar filas en una tabla, ClickHouse escribe bloques de datos en el directorio del disco para que puedan restaurarse cuando se reinicie el servidor.

Si el servidor se reinicia incorrectamente, el bloque de datos en el disco puede perderse o dañarse. En este caso, es posible que deba eliminar manualmente el archivo con datos dañados.

[Artículo Original](https://clickhouse.tech/docs/en/operations/table_engines/join/) <!--hide-->
