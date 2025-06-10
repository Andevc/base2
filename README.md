# 📘 PostgreSQL: Procedimientos, Cursores y Triggers

Este documento explica cómo crear y utilizar **procedimientos almacenados**, **cursores** y **triggers** en PostgreSQL usando `PL/pgSQL`.

---

## 📌 Índice

- [1. Procedimientos](#1-procedimientos)
- [2. Cursores](#2-cursores)
- [3. Cursores Almacenados en Procedimientos](#3-cursores-almacenados-en-procedimientos)
- [4. Triggers (Disparadores)](#4-triggers-disparadores)

---

## 1. 📋 Procedimientos

Los procedimientos permiten ejecutar bloques de código reutilizables que pueden modificar datos, sin retornar valores directamente.

### ✅ Estructura

```sql
CREATE OR REPLACE PROCEDURE nombre_procedimiento(param1 tipo, param2 tipo)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Lógica del procedimiento
END;
$$;
```
🧪 Ejemplo
```sql
CREATE OR REPLACE PROCEDURE insertar_usuario(nombre TEXT, edad INT)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO usuarios(nombre, edad) VALUES (nombre, edad);
END;
$$;
```
▶️ Llamar al procedimiento
```sql
CALL insertar_usuario('Carlos', 25);
```
## 2. 🔁 Cursores
Un cursor es un puntero que permite recorrer fila por fila el resultado de una consulta. Es útil cuando necesitas procesar cada fila individualmente (por ejemplo, con lógica condicional o cálculos complejos).
  🧠 Existen cursores implícitos y explícitos. Los explícitos te permiten un control total con OPEN, FETCH, CLOSE.

✅ Estructura
```sql
DECLARE cursor_nombre CURSOR FOR
SELECT * FROM tabla;
```
🧪 Ejemplo
```sql
DO $$
DECLARE
    rec RECORD;
    cur CURSOR FOR SELECT nombre, edad FROM usuarios;
BEGIN
    OPEN cur;
    LOOP
        FETCH cur INTO rec;
        EXIT WHEN NOT FOUND;
        RAISE NOTICE 'Nombre: %, Edad: %', rec.nombre, rec.edad;
    END LOOP;
    CLOSE cur;
END;
$$;
```
### 2.1 📂 Cursores Almacenados en Procedimientos
Puedes usar cursores dentro de procedimientos para recorrer los resultados de una consulta y ejecutar acciones personalizadas por cada fila obtenida.
- 🧠 Los cursores dentro de procedimientos siguen la misma estructura que en bloques DO, pero encapsulados.

🧪 Ejemplo
```sql
CREATE OR REPLACE PROCEDURE listar_usuarios()
LANGUAGE plpgsql
AS $$
DECLARE
    usuario_cursor CURSOR FOR SELECT * FROM usuarios;
    fila RECORD;
BEGIN
    OPEN usuario_cursor;
    LOOP
        FETCH usuario_cursor INTO fila;
        EXIT WHEN NOT FOUND;
        RAISE NOTICE 'Usuario: %, Edad: %', fila.nombre, fila.edad;
    END LOOP;
    CLOSE usuario_cursor;
END;
$$;
```
🔧 Cláusulas de los cursores 

| Opción / Cláusula       | Descripción                                                              |
|-------------------------|--------------------------------------------------------------------------|
| `DECLARE cursor_name`   | Declara el cursor con un nombre identificador.                          |
| `FOR SELECT ...`        | Define la consulta que recorrerá el cursor.                             |
| `OPEN cursor_name`      | Abre el cursor para empezar a recorrer sus filas.                       |
| `FETCH [IN] cursor`     | Extrae la siguiente fila del cursor (también puede usarse `FETCH NEXT`).|
| `MOVE`                  | Avanza el cursor sin recuperar datos (útil para saltos).                |
| `CLOSE cursor_name`     | Cierra el cursor y libera recursos.                                     |
| `WITH HOLD`             | Permite que el cursor siga disponible después de `COMMIT`.              |
| `SCROLL`                | Permite moverse hacia delante y atrás entre las filas.                  |
| `NO SCROLL`             | Solo permite moverse hacia adelante (por defecto).                      |

## 3. 🚨 Triggers (Disparadores)
Un trigger (disparador) es una función especial que se ejecuta automáticamente cuando ocurre un evento (como INSERT, UPDATE o DELETE) en una tabla.
- 🧠 Se componen de dos partes: una función trigger (que define qué hacer) y un trigger que la asocia a un evento en una tabla.

✅ Paso 1: Crear función del trigger
```sql
CREATE OR REPLACE FUNCTION log_insert()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO log_usuarios(accion, nombre)
    VALUES ('INSERT', NEW.nombre);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```
✅ Paso 2: Crear el trigger
```sql
CREATE TRIGGER trigger_log_insert
AFTER INSERT ON usuarios
FOR EACH ROW
EXECUTE FUNCTION log_insert();
```

🔧 Cláusulas y Variantes del Trigger
| Cláusula	                           | Descripción                                           |
|--------------------------------------|-------------------------------------------------------|
| `BEFORE` / `AFTER`	                     | Ejecuta la función antes o después del evento.        |
| `INSTEAD OF`	                         | Usado en vistas. Sustituye el evento por la función.  | 
| `INSERT` / `UPDATE` / `DELETE` / `TRUNCATE`	 | Eventos que pueden activar el trigger.                |
| `FOR EACH ROW`	                       | Ejecuta el trigger por cada fila afectada.            |
| `FOR EACH STATEMENT`                 | Ejecuta el trigger una vez por operación, sin importar el número de filas. |
| `WHEN` (condición)	                   | Condición opcional para ejecutar el trigger solo si se cumple.             |

| Cláusula                | Uso principal                                                                 |
|-------------------------|------------------------------------------------------------------------------|
| `BEFORE`                | Ejecuta la función **antes** de que ocurra el evento.                        |
| `AFTER`                 | Ejecuta la función **después** de que el evento se haya realizado.           |
| `INSTEAD OF`            | Sustituye el evento en vistas, porque no se puede modificar directamente.   |
| `INSERT`                | Se activa cuando se **inserta** una nueva fila.                              |
| `UPDATE`                | Se activa cuando se **modifica** una fila existente.                         |
| `DELETE`                | Se activa cuando se **elimina** una fila.                                    |
| `TRUNCATE`              | Se activa cuando se ejecuta `TRUNCATE` (elimina todos los registros).        |
| `FOR EACH ROW`          | Ejecuta el trigger **por cada fila** afectada por el evento.                 |
| `FOR EACH STATEMENT`    | Ejecuta el trigger **una sola vez** por sentencia, sin importar filas.       |
| `WHEN (condición)`      | Ejecuta el trigger **solo si** se cumple la condición especificada.         |

