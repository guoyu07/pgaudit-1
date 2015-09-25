pgAudit
=======
Sistema de auditoria para tablas postgreSQL basada en triggers.

Instalación
------------
Basta con ejecutar el script pgaudit.sql dentro de la base de datos a auditar, este proceso generara un esquema y dos funciones.

uso
---
Para auditar una tabla se hace uso de la función **pgaudit.table(schema, table)**, la cual se encarga de crear la tabla de auditoria *pgaudit.schema$table* dentro del esquema pgaudit.

| Nombre        | Descripción                                                                                                       |
|---------------|-------------------------------------------------------------------------------------------------------------------|
| id            | Autoincremental de la tabla                                                                                       |
| register_date | Fecha de registro del evento                                                                                      |
| user_db       | Usuario de la base de datos que modifico el registro                                                              |
| log_id        | Id de la tabla de log de la aplicación                                                                            |
| comando       | Operación realizada sobre el registro *(I,U,D)*                                                                   |
| old           | Registro anterior a la modificación realizada, si la modificación fue de tipo I este registro se encuentra vacio  |
| new           | Registro posterior a la modificación realizada, si la modificación fue de tipo D este registro se encuentra vacio |

Si se realiza una operación DDL el sistema registra automaticamente los campos: id, register_date, user_db, comando, old y new. Si se desea realizar auditoria directa sobre la aplicación se debe hacer uso del campo log_id para referenciar la tabla que se encargara de mantener la información de ingreso del usuario al sistema, el registro de esta información se debe realizar implementando la función **pgaudit.trail(idLog)** dentro de la misma aplicación.

```php
$db->exec("SELECT pgaudit.trail($_SESSION['log'])");
```

Las operaciones soportadas por pgAudit son: INSERT, UPDATE y DELETE, cada una de estas operaciones crea un registro en la tabla de auditoria.

Un registro se puede recuperar facilmente desde la tabla de auditoria con tan solo usar los campos old y new.

```sql
INSERT INTO public.usuario
SELECT (old).id, (old).nombre, (old).email, (old).password
FROM pgaudit.public$usuario
WHERE id = 1;
```

El manejo de registros como tipo de dato es tan versatil y poderoso que se pueden realizar consultas gracias a ellos.

```sql
SELECT (new).id
FROM pgaudit.public$usuario
WHERE (new).nombre = 'admin';
```