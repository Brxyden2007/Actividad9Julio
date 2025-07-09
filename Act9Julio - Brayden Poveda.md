### ACTIVIDAD MySQL

Entrega MARKDOWN
Pasarlo con PROCEDIMIENTOS Y EVENT0S

## Actividad
Haciendo uso de las siguientes tablas para la base de datos de pizza realice los siguientes ejercicios de Events centrados en el uso de ON COMPLETION PRESERVE y ON COMPLETION NOT PRESERVE :

### Tablas:

```bash
CREATE TABLE IF NOT EXISTS resumen_ventas (
fecha       DATE      PRIMARY KEY,
total_pedidos INT,
total_ingresos DECIMAL(12,2),
creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

CREATE TABLE IF NOT EXISTS alerta_stock (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  ingrediente_id  INT UNSIGNED NOT NULL,
  stock_actual    INT NOT NULL,
  fecha_alerta    DATETIME NOT NULL,
  creado_en DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (ingrediente_id) REFERENCES ingrediente(id)
);

## Creacion de tabla ingrediente:

CREATE TABLE IF NOT EXISTS ingrediente (
  id              INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  nombre          VARCHAR(100) NOT NULL,
  tipo            VARCHAR(50),
  unidad_medida   VARCHAR(20) NOT NULL, 
  stock_actual    DECIMAL(10,2) NOT NULL DEFAULT 0.00,
  stock_minimo    DECIMAL(10,2) DEFAULT 0.00,
  costo_unitario  DECIMAL(10,2),
  proveedor       VARCHAR(100),
  fecha_caducidad DATE,
  creado_en       DATETIME DEFAULT CURRENT_TIMESTAMP
);

## Creacion tabla ventas (para resumen_ventas)

CREATE TABLE IF NOT EXISTS ventas (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  fecha_venta     DATETIME NOT NULL,
  total           DECIMAL(12,2) NOT NULL,
  creado_en       DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS log_eventos (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  nombre_evento   VARCHAR(100),
  descripcion     TEXT,
  ejecutado_en    DATETIME DEFAULT CURRENT_TIMESTAMP
);


### Puntos:

1. Resumen Diario Único : crear un evento que genere un resumen de ventas una sola vez al finalizar el día de ayer y luego se elimine automáticamente llamado ev_resumen_diario_unico.

2. Resumen Semanal Recurrente: cada lunes a las 01:00 AM, generar el total de pedidos e ingresos de la semana pasada, manteniendo el evento para que siga ejecutándose cada semana llamado ev_resumen_semanal.

3. Alerta de Stock Bajo Única: en un futuro arranque del sistema (requerimiento del sistema), generar una única pasada de alertas (alerta_stock) de ingredientes con stock < 5, y luego autodestruir el evento.

4. Monitoreo Continuo de Stock: cada 30 minutos, revisar ingredientes con stock < 10 e insertar alertas en alerta_stock, dejando el evento activo para siempre llamado ev_monitor_stock_bajo.

5. Limpieza de Resúmenes Antiguos: una sola vez, eliminar de resumen_ventas los registros con fecha anterior a hace 365 días y luego borrar el evento llamado ev_purgar_resumen_antiguo.

-------------------------------------------------------------------------------------------------------


#### |----- Punto 1 -----|

-------------------------------
Solucion Procedimiento:
-----------------------------
DELIMITER //

CREATE EVENT ev_resumen_diario_unico
ON SCHEDULE
AT CURRENT_DATE + INTERVAL 1 DAY + INTERVAL 10 MINUTE
ON COMPLETION NOT PRESERVE
DO
BEGIN
INSERT INTO resumen_diario_ventas (fecha, total_ventas, cantidad_ventas, creado_en)
SELECT
CURDATE() - INTERVAL 1 DAY AS fecha,
COALESCE(SUM(total), 0) AS total_ingresos
FROM ventas
WHERE DATE(fecha_venta) = CURDATE() - INTERVAL 1 DAY;
END //

DELIMITER ; 

-----------------------------------------------------------
### Mostrar el evento:

SHOW CREATE EVENT ev_resumen_diario_unico;

-----------------------------------------------------------

------------------------------------///////////////////----------------------------------------------

#### |----- Punto 2 -----|

-------------------------------
Solucion Procedimiento:
---------------------------------------

DELIMITER //

CREATE EVENT ev_resumen_semanal
ON SCHEDULE EVERY 1 WEEK
STARTS '2025-07-14 00:10:00' 
DO
BEGIN
  DECLARE semana_inicio DATE;
  SET semana_inicio = DATE_SUB(CURRENT_DATE, INTERVAL WEEKDAY(CURRENT_DATE) + 7 DAY);

  INSERT INTO resumen_ventas (fecha, total_pedidos, total_ingresos)
  SELECT
    semana_inicio AS fecha,
    COUNT(*) AS total_pedidos,
    COALESCE(SUM(total), 0) AS total_ingresos
  FROM ventas
  WHERE fecha_venta >= semana_inicio
    AND fecha_venta < DATE_ADD(semana_inicio, INTERVAL 7 DAY);
END //

DELIMITER ;

Explicacion: Tuve un dilema/duda, debido a que el primer punto pedia diariamente hacerlo, y el otro semanalmente, al tratar de hacerlo tuve que utilizar el STARTS para indicar la fecha en la que empezaria a correr, en este caso, el Lunes 14.

------------------------------------------------------------------------------------------------------
### Mostrar el evento:

SHOW CREATE EVENT ev_resumen_semanal;

--------------------------------------------///////////////-------------------------------------------


#### |----- Punto 3 -----|

-------------------------------
Solucion Procedimiento:
---------------------------------------

DELIMITER //

CREATE EVENT ev_alerta_stock
ON SCHEDULE AT CURRENT_TIMESTAMP
ON COMPLETION NOT PRESERVE
DO
BEGIN
  INSERT INTO alerta_stock (ingrediente_id, stock_actual, fecha_alerta)
  SELECT id, stock_actual, NOW()
  FROM ingrediente
  WHERE stock_actual < 5;

  INSERT INTO log_eventos (nombre_evento, descripcion)
  VALUES (
    'ev_alerta_stock',
    'Se ejecuta el evento y luego se elimina'
  );
END //

DELIMITER ;

------------------------------------
### Mostrar evento eliminado:

SELECT id, nombre_evento, descripcion, ejecutado_en FROM log_eventos;
------------------------------------

#### |----- Punto 4 -----|

-------------------------------
Solucion Procedimiento:
---------------------------------------

DELIMITER //

CREATE EVENT ev_monitor_stock_bajo
ON SCHEDULE EVERY 30 MINUTE
STARTS CURRENT_TIMESTAMP
ON COMPLETION PRESERVE
DO
BEGIN
  INSERT INTO alerta_stock (ingrediente_id, stock_actual, fecha_alerta)
  SELECT id, stock_actual, NOW()
  FROM ingrediente
  WHERE stock_actual < 10;
END //

DELIMITER ;

-----------------------------------
### Ver el evento:
----------------------------------

SHOW CREATE EVENT ev_monitor_stock_bajo;


------------------------------------------

#### |----- Punto 5 -----|


-------------------------------
Solucion Procedimiento:
---------------------------------------

DELIMITER //

CREATE EVENT ev_purgar_resumen_antiguo
ON SCHEDULE AT CURRENT_TIMESTAMP
ON COMPLETION NOT PRESERVE
DO
BEGIN
  DELETE FROM resumen_ventas
  WHERE fecha < CURDATE() - INTERVAL 365 DAY;

  INSERT INTO log_eventos (nombre_evento, descripcion)
  VALUES (
    'ev_purgar_resumen_antiguo',
    'Evento ejecutado para borrar registro antiguo (365 dias)'
  );
END //

DELIMITER ;

-----------------------------------
### Ver el evento:
----------------------------------

SELECT id, nombre_evento, descripcion, ejecutado_en FROM log_eventos WHERE nombre_evento = 'ev_purgar_resumen_antiguo';




