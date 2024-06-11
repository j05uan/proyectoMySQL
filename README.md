# proyectoMySQL

## Tablas creadas
~~~mysql
show tables;
+-------------------+
| Tables_in_taller  |
+-------------------+
| cargo             |
| cita              |
| ciudad            |
| cliente           |
| detalleFactura    |
| empleado          |
| factura           |
| inventario        |
| marca             |
| modelo            |
| ordenCompra       |
| ordenDetalle      |
| pais              |
| pieza             |
| proveedor         |
| reparacion        |
| reparacionPieza   |
| servicio          |
| sucursal          |
| telefonoCliente   |
| telefonoEmpleado  |
| telefonoProveedor |
| tipoTelefono      |
| ubicacionSucursal |
| vehiculo          |
+-------------------+
~~~

1. Obtener el historial de reparaciones de un vehículo específico
~~~mysql
SELECT r.*, v.placa AS PlacaVehiculo,s.nombre AS NombreServicio, e.nombre AS NombreEmpleado
	FROM reparacion r
	JOIN servicio s ON r.idServicio = s.idServicio
	JOIN vehiculo v ON v.idVehiculo = r.idVehiculo
	JOIN empleado e ON e.idEmpleado = r.idEmpleado
	WHERE v.placa = 'ABC123';
~~~
2. Calcular el costo total de todas las reparaciones realizadas por un empleado
específico en un período de tiempo
~~~mysql
  SELECT SUM(r.costoTotal) AS costoTotalReparaciones
	FROM reparacion r
	JOIN empleado e ON r.idEmpleado = e.idEmpleado
	WHERE e.nombre = 'luis' 
	AND r.fecha BETWEEN '2022-12-31' AND '2023-12-31';

+------------------------+
| costoTotalReparaciones |
+------------------------+
|                   NULL |
+------------------------+


~~~
3. Listar todos los clientes y los vehículos que poseen 
~~~mysql

~~~
