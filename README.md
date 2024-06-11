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
+--------------+------------+------------+------------+------------+------------+------------------+---------------+---------------+------------------+----------------+
| idReparacion | idVehiculo | idEmpleado | idServicio | fecha      | costoTotal | descripcion      | duracionHoras | PlacaVehiculo | NombreServicio   | NombreEmpleado |
+--------------+------------+------------+------------+------------+------------+------------------+---------------+---------------+------------------+----------------+
|            1 |          1 |          1 |          1 | 2023-05-01 |         35 | Cambio de aceite | 03:00         | ABC123        | Cambio de aceite | Luis           |
+--------------+------------+------------+------------+------------+------------+------------------+---------------+---------------+------------------+----------------+

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
|                     35 |
+------------------------+
~~~
3. Listar todos los clientes y los vehículos que poseen 
~~~mysql
SELECT c.nombre, c.apellidos, ma.marca, mo.modelo , v.placa 
     FROM cliente c
     LEFT JOIN vehiculo v ON c.idCliente = v.idCliente
     LEFT JOIN modelo mo ON v.idModelo = mo.idModelo
     LEFT JOIN marca ma ON mo.idMarca = ma.idMarca;
+--------+-----------+-----------+---------+--------+
| nombre | apellidos | marca     | modelo  | placa  |
+--------+-----------+-----------+---------+--------+
| Juan   | Perez     | Toyota    | Corolla | ABC123 |
| Maria  | Garcia    | Ford      | Fiesta  | DEF456 |
| Carlos | Lopez     | Chevrolet | Cruze   | GHI789 |
+--------+-----------+-----------+---------+--------+
~~~

4. Obtener la cantidad de piezas en inventario para cada pieza
~~~mysql	
	SELECT p.nombre, SUM(i.cantidad) AS cantidad_total
	FROM inventario i
	JOIN pieza p ON p.idPieza = i.idPieza
	GROUP BY p.nombre;
+--------------------+----------------+
| nombre             | cantidad_total |
+--------------------+----------------+
| Filtro de aceite   |             50 |
| Bujía              |            100 |
| Pastillas de freno |            200 |
+--------------------+----------------+
~~~

5. Obtener las citas programadas para un día específico
~~~mysql
	
	SELECT c.*, r.descripcion AS reparacion FROM cita c
	JOIN reparacion r ON c.idReparacion = r.idReparacion
	WHERE DATE(fechaHora) = '2023-06-04';
+--------+--------------+---------------------+------------------+
| idCita | idReparacion | fechaHora           | reparacion       |
+--------+--------------+---------------------+------------------+
|      1 |            1 | 2023-06-04 10:00:00 | Cambio de aceite |
|      2 |            2 | 2023-06-04 11:00:00 | Alineación       |
|      3 |            3 | 2023-06-04 12:00:00 | Balanceo         |
+--------+--------------+---------------------+------------------+

~~~

6. Generar una factura para un cliente específico en una fecha determinada
~~~mysql
	SELECT f.*, c.nombre , df.precioFinal, r.descripcion AS reparacion
	FROM factura f
	JOIN cliente c ON c.idCliente = f.idCliente
	JOIN detalleFactura df ON f.idFactura = df.idFactura
	JOIN reparacion r ON df.idReparacion = r.idReparacion
	WHERE  DATE(f.fecha)='2023-06-05' 
	AND c.nombre = 'Juan';
+-----------+-----------+------------+-------+--------+-------------+------------------+
| idFactura | idCliente | fecha      | total | nombre | precioFinal | reparacion       |
+-----------+-----------+------------+-------+--------+-------------+------------------+
|         1 |         1 | 2023-06-05 |    35 | Juan   |          35 | Cambio de aceite |
+-----------+-----------+------------+-------+--------+-------------+------------------+

~~~

7. Listar todas las órdenes de compra y sus detalles
~~~mysql
	SELECT OD.*, OC.*
	FROM ordenDetalle OD
	JOIN ordenCompra OC ON OD.idOrdenCompra = OC.idOrdenCompra;
+---------------+---------+----------+--------+---------------+-------------+------------+------------+-------+
| idOrdenCompra | idPieza | cantidad | precio | idOrdenCompra | idProveedor | idEmpleado | fecha      | total |
+---------------+---------+----------+--------+---------------+-------------+------------+------------+-------+
|             1 |       1 |       10 |     10 |             1 |           1 |          1 | 2023-06-01 |   200 |
|             1 |       2 |       20 |      5 |             1 |           1 |          1 | 2023-06-01 |   200 |
|             2 |       3 |       30 |      3 |             2 |           2 |          2 | 2023-06-02 |   300 |
+---------------+---------+----------+--------+---------------+-------------+------------+------------+-------+
~~~
8. Obtener el costo total de piezas utilizadas en una reparación específica
~~~mysql
	SELECT SUM(od.precio * rp.cantidad) AS costoTotal
	FROM reparacionPieza rp
	JOIN ordenDetalle od ON od.idPieza = rp.idPieza
	JOIN pieza p ON rp.idPieza = p.idPieza
	WHERE rp.idReparacion = 1;
+------------+
| costoTotal |
+------------+
|         10 |
+------------+

~~~
9. Obtener el inventario de piezas que necesitan ser reabastecidas (cantidad
menor que un umbral)
~~~mysql
	SELECT p.nombre AS pieza, i.cantidad
	FROM inventario i
	JOIN pieza p ON i.idPieza = p.idPieza
	WHERE i.cantidad < 10;
~~~

10. Obtener la lista de servicios más solicitados en un período específico
~~~mysql
SELECT s.nombre AS servicio, COUNT(r.idServicio) AS cantidadSolicitada
	FROM reparacion r
	JOIN servicio s ON r.idServicio = s.idServicio
	WHERE r.fecha BETWEEN '2023-01-01' AND '2023-12-31'
	GROUP BY r.idServicio
	ORDER BY cantidadSolicitada DESC;
~~~
11. Obtener el costo total de reparaciones para cada cliente en un período
específico
~~~mysql
	SELECT c.nombre AS cliente, SUM(r.costoTotal) AS costoTotal
	FROM reparacion r
	JOIN vehiculo v ON r.idVehiculo = v.idVehiculo
	JOIN cliente c ON v.idCliente = c.idCliente
	WHERE r.fecha BETWEEN '2023-01-01' AND '2023-12-31'
	GROUP BY c.idCliente;
+---------+------------+
| cliente | costoTotal |
+---------+------------+
| Juan    |         35 |
| Maria   |         55 |
| Carlos  |         45 |
+---------+------------+
~~~

12. Listar los empleados con mayor cantidad de reparaciones realizadas en un
período específico
~~~mysql
	SELECT e.nombre AS empleado, COUNT(r.idReparacion) AS cantidadReparaciones
	FROM reparacion r
	JOIN empleado e ON r.idEmpleado = e.idEmpleado
	WHERE r.fecha BETWEEN '2023-01-01' AND '2023-12-31'
	GROUP BY e.idEmpleado
	ORDER BY cantidadReparaciones DESC;
+----------+----------------------+
| empleado | cantidadReparaciones |
+----------+----------------------+
| Luis     |                    1 |
| Ana      |                    1 |
| Pedro    |                    1 |
+----------+----------------------+
~~~
13. Obtener las piezas más utilizadas en reparaciones durante un período
específico
~~~mysql
	SELECT p.nombre AS pieza, SUM(rp.cantidad) AS cantidadUsada
	FROM reparacionPieza rp
	JOIN pieza p ON rp.idPieza = p.idPieza
	JOIN reparacion r ON rp.idReparacion = r.idReparacion
	WHERE r.fecha BETWEEN '2023-01-01' AND '2023-12-31'
	GROUP BY p.idPieza
	ORDER BY cantidadUsada DESC;
+--------------------+---------------+
| pieza              | cantidadUsada |
+--------------------+---------------+
| Bujía              |             4 |
| Pastillas de freno |             2 |
| Filtro de aceite   |             1 |
+--------------------+---------------+
~~~

14. Calcular el promedio de costo de reparaciones por vehículo
~~~mysql
	SELECT v.placa, AVG(r.costoTotal) AS promedioCosto
	FROM reparacion r
	JOIN vehiculo v ON r.idVehiculo = v.idVehiculo
	GROUP BY v.idVehiculo;
+--------+---------------+
| placa  | promedioCosto |
+--------+---------------+
| ABC123 |            35 |
| DEF456 |            55 |
| GHI789 |            45 |
+--------+---------------+
~~~

15. Obtener el inventario de piezas por proveedor
~~~mysql
	SELECT p.nombre AS proveedor, pi.nombre AS pieza, SUM(i.cantidad) AS cantidad
	FROM inventario i
	JOIN pieza pi ON i.idPieza = pi.idPieza
	JOIN ordenDetalle od ON pi.idPieza = od.idPieza
	JOIN ordenCompra oc ON od.idOrdenCompra = oc.idOrdenCompra
	JOIN proveedor p ON oc.idProveedor = p.idProveedor
	GROUP BY p.idProveedor, pi.idPieza;
+------------------+--------------------+----------+
| proveedor        | pieza              | cantidad |
+------------------+--------------------+----------+
| Repuestos Toyota | Filtro de aceite   |       50 |
| Repuestos Toyota | Bujía              |      100 |
| Repuestos Ford   | Pastillas de freno |      200 |
+------------------+--------------------+----------+
~~~
16. Listar los clientes que no han realizado reparaciones en el último año
~~~mysql
	SELECT c.nombre AS cliente
	FROM cliente c
	LEFT JOIN vehiculo v ON c.idCliente = v.idCliente
	LEFT JOIN reparacion r ON v.idVehiculo = r.idVehiculo
	WHERE r.fecha IS NULL OR r.fecha < DATE_SUB(CURDATE(), INTERVAL 1 YEAR);
+---------+
| cliente |
+---------+
| Juan    |
| Maria   |
| Carlos  |
+---------+
~~~

17. Obtener las ganancias totales del taller en un período específico
~~~mysql
	SELECT SUM(f.total) AS gananciasTotales
	FROM factura f
	WHERE f.fecha BETWEEN '2023-01-01' AND '2023-12-31';
+------------------+
| gananciasTotales |
+------------------+
|              135 |
+------------------+
~~~
18. Listar los empleados y el total de horas trabajadas en reparaciones en un
período específico (asumiendo que se registra la duración de cada reparación)
~~~mysql
	SELECT e.nombre AS empleado, SUM(r.duracionHoras) AS horasTrabajadas
	FROM reparacion r
	JOIN empleado e ON r.idEmpleado = e.idEmpleado
	WHERE r.fecha BETWEEN '2023-01-01' AND '2023-12-31'
	GROUP BY e.idEmpleado;
+----------+-----------------+
| empleado | horasTrabajadas |
+----------+-----------------+
| Luis     |               3 |
| Ana      |               1 |
| Pedro    |               0 |
+----------+-----------------+
~~~

19. Obtener el listado de servicios prestados por cada empleado en un período
específico
~~~mysql
	SELECT e.nombre AS empleado, s.nombre AS servicio, COUNT(r.idReparacion) AS cantidadServicios
	FROM reparacion r
	JOIN empleado e ON r.idEmpleado = e.idEmpleado
	JOIN servicio s ON r.idServicio = s.idServicio
	WHERE r.fecha BETWEEN '2023-01-01' AND '2023-12-31'
	GROUP BY e.idEmpleado, s.idServicio;
+----------+------------------+-------------------+
| empleado | servicio         | cantidadServicios |
+----------+------------------+-------------------+
| Luis     | Cambio de aceite |                 1 |
| Ana      | Alineación       |                 1 |
| Pedro    | Balanceo         |                 1 |
+----------+------------------+-------------------+
~~~

##SUBCONSULTAS

1. Obtener el cliente que ha gastado más en reparaciones durante el último año.
 ~~~mysql
  SELECT c.nombre, SUM(r.costoTotal) AS gastoTotal from cliente c
  JOIN vehiculo v ON c.idCliente = v.idCliente
  JOIN reparacion r ON v.idVehiculo = r.idVehiculo
  WHERE r.fecha BETWEEN DATE_SUB(CURDATE(), INTERVAL 1 YEAR) AND CURDATE()
  GROUP BY c.idCliente
  ORDER BY gastoTotal DESC
  LIMIT 1;
+--------+------------+
| nombre | gastoTotal |
+--------+------------+
| Carlos |        500 |
+--------+------------+
~~~

2. Obtener la pieza más utilizada en reparaciones durante el último mes
 ~~~mysql
  SELECT p.nombre, SUM(rp.cantidad) AS cantidadUsada
  FROM pieza p
  JOIN reparacionPieza rp ON p.idPieza = rp.idPieza
  JOIN reparacion r ON rp.idReparacion = r.idReparacion
  WHERE r.fecha BETWEEN DATE_SUB(CURDATE(), INTERVAL 1 MONTH) AND CURDATE()
  GROUP BY p.idPieza
  ORDER BY cantidadUsada DESC
  LIMIT 1;

 ~~~

3. Obtener los proveedores que suministran las piezas más caras
 ~~~mysql
  SELECT pr.nombre AS proveedor, p.nombre AS pieza, od.precio
  FROM proveedor pr
  JOIN ordenCompra oc ON pr.idProveedor = oc.idProveedor
  JOIN ordenDetalle od ON oc.idOrdenCompra = od.idOrdenCompra
  JOIN pieza p ON od.idPieza = p.idPieza
  ORDER BY od.precio DESC
  LIMIT 1;
+------------------+------------------+--------+
| proveedor        | pieza            | precio |
+------------------+------------------+--------+
| Repuestos Toyota | Filtro de aceite |     10 |
+------------------+------------------+--------+
~~~
4. Listar las reparaciones que no utilizaron piezas específicas durante el último
año
 ~~~mysql
  SELECT r.idReparacion, r.descripcion
  FROM reparacion r
  LEFT JOIN reparacionPieza rp ON r.idReparacion = rp.idReparacion AND rp.idPieza = 1
  WHERE rp.idPieza IS NULL AND r.fecha BETWEEN DATE_SUB(CURDATE(), INTERVAL 1 YEAR) AND CURDATE();
+--------------+--------------------------------+
| idReparacion | descripcion                    |
+--------------+--------------------------------+
|            6 | Reemplazo de frenos delanteros |
+--------------+--------------------------------+
 ~~~
## PROCEDIMIENTOS

1 Crear un procedimiento almacenado para insertar una nueva reparación.
 ~~~mysql
DELIMITER //
CREATE PROCEDURE insertarReparacion (
    IN p_idVehiculo INT,
    IN p_idEmpleado INT,
    IN p_idServicio INT,
    IN p_fecha DATE,
    IN p_costoTotal FLOAT,
    IN p_descripcion TEXT )
BEGIN
    INSERT INTO reparacion (idVehiculo, idEmpleado, idServicio, fecha, costoTotal, descripcion)
    VALUES (p_idVehiculo, p_idEmpleado, p_idServicio, p_fecha, p_costoTotal, p_descripcion);
END //
DELIMITER ;

CALL insertarReparacion(3, 2, 1, '2024-06-11', 500.00, 'Reemplazo de frenos delanteros');
 ~~~
2 Crear un procedimiento almacenado para actualizar el inventario de una pieza.
 ~~~mysql
DELIMITER //
CREATE PROCEDURE actualizarInventario (
    IN p_idPieza INT,
    IN p_nuevaCantidad INT )
BEGIN
    UPDATE inventario
    SET cantidad = p_nuevaCantidad
    WHERE idPieza = p_idPieza;
END //
DELIMITER ;

CALL actualizarInventario(1, 150);
 ~~~
3 Crear un procedimiento almacenado para eliminar una cita
 ~~~mysql
DELIMITER //
CREATE PROCEDURE eliminarCita (
    IN p_idCita INT )
BEGIN
    DELETE FROM cita
    WHERE idCita = p_idCita;
END //
DELIMITER ;
CALL eliminarCita (1);
 ~~~
4 Crear un procedimiento almacenado para generar una factura
 ~~~mysql
DELIMITER //
CREATE PROCEDURE generarFactura (
    IN p_idCliente INT,
    IN p_fecha DATE,
    IN p_total FLOAT)
BEGIN
    INSERT INTO factura (idCliente, fecha, total)
    VALUES (p_idCliente, p_fecha, p_total);
END //
DELIMITER ;
CALL generarFactura (1,'2024-01-23',25);
 ~~~
5 Crear un procedimiento almacenado para obtener el historial de reparaciones de un vehículo
 ~~~
DELIMITER //
CREATE PROCEDURE obtenerHistorialReparaciones (
    IN p_idVehiculo INT )
BEGIN
    SELECT v.placa, r.idReparacion, r.fecha, r.costoTotal, r.descripcion, s.nombre AS servicio, e.nombre AS empleado
    FROM reparacion r
    JOIN servicio s ON r.idServicio = s.idServicio
    JOIN vehiculo v ON v.idVehiculo = r.idVehiculo
    JOIN empleado e ON r.idEmpleado = e.idEmpleado
    WHERE r.idVehiculo = p_idVehiculo;
END //
DELIMITER ;
CALL obtenerHistorialReparaciones (1);
 ~~~
6 Crear un procedimiento almacenado para calcular el costo total de reparaciones de un cliente en un período
 ~~~mysql
DELIMITER //
CREATE PROCEDURE costoTotalReparacionesCliente (
    IN p_idCliente INT,
    IN p_fechaInicio DATE,
    IN p_fechaFin DATE )
BEGIN
    SELECT SUM(r.costoTotal) AS costoTotal
    FROM reparacion r
    JOIN vehiculo v ON r.idVehiculo = v.idVehiculo
    WHERE v.idCliente = p_idCliente AND r.fecha BETWEEN p_fechaInicio AND p_fechaFin;
END //
DELIMITER ;
CALL 
 ~~~

7 Crear un procedimiento almacenado para obtener la lista de vehículos que requieren mantenimiento basado en el kilometraje.


8 Crear un procedimiento almacenado para insertar una nueva orden de compra
 ~~~mysql
DELIMITER //
CREATE PROCEDURE insertarOrdenCompra (
    IN p_idProveedor INT,
    IN p_idEmpleado INT,
    IN p_fecha DATE,
    IN p_total FLOAT
)
BEGIN
    INSERT INTO ordenCompra (idProveedor, idEmpleado, fecha, total)
    VALUES (p_idProveedor, p_idEmpleado, p_fecha, p_total);
END //
DELIMITER ;
 ~~~
9 Crear un procedimiento almacenado para actualizar los datos de un cliente
 ~~~mysql
DELIMITER //
CREATE PROCEDURE actualizarCliente (
    IN p_idCliente INT,
    IN p_nombre VARCHAR(255),
    IN p_apellidos VARCHAR(255),
    IN p_direccion VARCHAR(255)
)
BEGIN
    UPDATE cliente
    SET nombre = p_nombre, apellidos = p_apellidos, direccion = p_direccion
    WHERE idCliente = p_idCliente;
END //
DELIMITER ;
 ~~~
10 Crear un procedimiento almacenado para obtener los servicios más solicitados en un período
 ~~~mysql
DELIMITER //
CREATE PROCEDURE serviciosMasSolicitados (
    IN p_fechaInicio DATE,
    IN p_fechaFin DATE
)
BEGIN
    SELECT s.nombre AS servicio, COUNT(r.idServicio) AS cantidadSolicitada
    FROM reparacion r
    JOIN servicio s ON r.idServicio = s.idServicio
    WHERE r.fecha BETWEEN p_fechaInicio AND p_fechaFin
    GROUP BY r.idServicio
    ORDER BY cantidadSolicitada DESC;
END //
DELIMITER ;
 ~~~
