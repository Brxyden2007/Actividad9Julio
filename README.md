# Actividad
Haciendo uso de las siguientes tablas para la base de datos de pizza realice los siguientes ejercicios de Events centrados en el uso de ON COMPLETION PRESERVE y ON COMPLETION NOT PRESERVE :
```bash
CREATE TABLE IF NOT EXISTS resumen_ventas (
fecha       DATE      PRIMARY KEY,
total_pedidos INT,
total_ingresos DECIMAL(12,2),
creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);
```
```bash
CREATE TABLE IF NOT EXISTS alerta_stock (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  ingrediente_id  INT UNSIGNED NOT NULL,
  stock_actual    INT NOT NULL,
  fecha_alerta    DATETIME NOT NULL,
  creado_en DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (ingrediente_id) REFERENCES ingrediente(id)
);
```

Resumen Diario Único : crear un evento que genere un resumen de ventas una sola vez al finalizar el día de ayer y luego se elimine automáticamente llamado ev_resumen_diario_unico.
Resumen Semanal Recurrente: cada lunes a las 01:00 AM, generar el total de pedidos e ingresos de la semana pasada, manteniendo el evento para que siga ejecutándose cada semana llamado ev_resumen_semanal.
Alerta de Stock Bajo Única: en un futuro arranque del sistema (requerimiento del sistema), generar una única pasada de alertas (alerta_stock) de ingredientes con stock < 5, y luego autodestruir el evento.
Monitoreo Continuo de Stock: cada 30 minutos, revisar ingredientes con stock < 10 e insertar alertas en alerta_stock, dejando el evento activo para siempre llamado ev_monitor_stock_bajo.
Limpieza de Resúmenes Antiguos: una sola vez, eliminar de resumen_ventas los registros con fecha anterior a hace 365 días y luego borrar el evento llamado ev_purgar_resumen_antiguo.
