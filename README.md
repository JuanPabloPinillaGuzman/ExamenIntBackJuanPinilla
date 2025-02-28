# ExamenIntBackJuanPinilla

Examen #1

1. Diagrama UML E-R: Un diagrama con entidades, atributos y relaciones:
2. Documentación: Un archivo PDF explicando el diseño de la base de datos:
[📄 Ver CampusCar Juan Pinilla.pdf](CampusCar%20Juan%20Pinilla.pdf)
3. SQL Script: entregar un script SQL para la creación de tablas y restricciones:

```sql
CREATE DATABASE IF NOT EXISTS ConcesionarioVehiculos;
USE ConcesionarioVehiculos;

-- Tabla de categorías de vehículos
CREATE TABLE CATEGORIA_VEHICULO (
    categoria_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL COMMENT 'Sedán/SUV/Pickup/etc.',
    descripcion TEXT
);

-- Tabla de proveedores
CREATE TABLE PROVEEDOR (
    proveedor_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    contacto VARCHAR(100),
    telefono VARCHAR(20),
    email VARCHAR(100),
    direccion TEXT
);

-- Tabla de sucursales
CREATE TABLE SUCURSAL (
    sucursal_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    direccion TEXT NOT NULL,
    telefono VARCHAR(20),
    email VARCHAR(100),
    horario VARCHAR(200)
);

-- Tabla de tipos de cliente
CREATE TABLE TIPO_CLIENTE (
    tipo_cliente_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL COMMENT 'Particular/Empresarial',
    porcentaje_descuento DECIMAL(5,2) DEFAULT 0.00
);

-- Tabla de vehículos
CREATE TABLE VEHICULO (
    vehiculo_id INT AUTO_INCREMENT PRIMARY KEY,
    vin VARCHAR(17) NOT NULL UNIQUE COMMENT 'Número de serie único',
    marca VARCHAR(50) NOT NULL,
    modelo VARCHAR(50) NOT NULL,
    anio INT NOT NULL,
    precio DECIMAL(12,2) NOT NULL,
    color VARCHAR(30),
    tipo_combustible VARCHAR(30),
    tipo_transmision VARCHAR(30),
    estado VARCHAR(10) NOT NULL COMMENT 'Nuevo/Usado',
    disponible BOOLEAN DEFAULT TRUE COMMENT 'True=Disponible, False=Vendido',
    proveedor_id INT,
    categoria_id INT,
    FOREIGN KEY (proveedor_id) REFERENCES PROVEEDOR(proveedor_id),
    FOREIGN KEY (categoria_id) REFERENCES CATEGORIA_VEHICULO(categoria_id)
);

-- Tabla de clientes
CREATE TABLE CLIENTE (
    cliente_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    telefono VARCHAR(20),
    email VARCHAR(100),
    direccion TEXT,
    fecha_registro DATE DEFAULT CURRENT_DATE,
    tipo_cliente_id INT,
    FOREIGN KEY (tipo_cliente_id) REFERENCES TIPO_CLIENTE(tipo_cliente_id)
);

-- Tabla de vendedores
CREATE TABLE VENDEDOR (
    vendedor_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    num_empleado VARCHAR(20) NOT NULL UNIQUE,
    fecha_contratacion DATE NOT NULL,
    comision_porcentaje DECIMAL(5,2) DEFAULT 0.00,
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES SUCURSAL(sucursal_id)
);

-- Tabla de mecánicos
CREATE TABLE MECANICO (
    mecanico_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    num_empleado VARCHAR(20) NOT NULL UNIQUE,
    especialidad VARCHAR(100),
    fecha_contratacion DATE NOT NULL,
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES SUCURSAL(sucursal_id)
);

-- Tabla de financiamiento
CREATE TABLE FINANCIAMIENTO (
    financiamiento_id INT AUTO_INCREMENT PRIMARY KEY,
    entidad_financiera VARCHAR(100) NOT NULL,
    tasa_interes DECIMAL(5,2) NOT NULL,
    plazo_meses INT NOT NULL,
    monto_financiado DECIMAL(12,2) NOT NULL,
    estado VARCHAR(20) DEFAULT 'Pendiente' COMMENT 'Aprobado/Pendiente/Rechazado'
);

-- Tabla de ventas
CREATE TABLE VENTA (
    venta_id INT AUTO_INCREMENT PRIMARY KEY,
    fecha_venta DATE NOT NULL,
    subtotal DECIMAL(12,2) NOT NULL,
    impuestos DECIMAL(12,2) NOT NULL,
    descuento DECIMAL(12,2) DEFAULT 0.00,
    total DECIMAL(12,2) NOT NULL,
    metodo_pago VARCHAR(30) NOT NULL,
    cliente_id INT NOT NULL,
    vendedor_id INT NOT NULL,
    financiamiento_id INT,
    FOREIGN KEY (cliente_id) REFERENCES CLIENTE(cliente_id),
    FOREIGN KEY (vendedor_id) REFERENCES VENDEDOR(vendedor_id),
    FOREIGN KEY (financiamiento_id) REFERENCES FINANCIAMIENTO(financiamiento_id)
);

-- Tabla de detalle de venta
CREATE TABLE DETALLE_VENTA (
    detalle_venta_id INT AUTO_INCREMENT PRIMARY KEY,
    venta_id INT NOT NULL,
    vehiculo_id INT NOT NULL,
    precio_venta DECIMAL(12,2) NOT NULL,
    descuento DECIMAL(12,2) DEFAULT 0.00,
    FOREIGN KEY (venta_id) REFERENCES VENTA(venta_id),
    FOREIGN KEY (vehiculo_id) REFERENCES VEHICULO(vehiculo_id),
    CONSTRAINT uq_vehiculo_venta UNIQUE (vehiculo_id) -- Un vehículo solo puede ser vendido una vez
);

-- Tabla de repuestos
CREATE TABLE REPUESTO (
    repuesto_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    precio DECIMAL(10,2) NOT NULL,
    cantidad_stock INT NOT NULL DEFAULT 0,
    codigo VARCHAR(50) NOT NULL UNIQUE
);

-- Tabla de mantenimiento
CREATE TABLE MANTENIMIENTO (
    mantenimiento_id INT AUTO_INCREMENT PRIMARY KEY,
    fecha_servicio DATE NOT NULL,
    tipo_servicio VARCHAR(50) NOT NULL COMMENT 'Preventivo/Correctivo',
    costo DECIMAL(10,2) NOT NULL,
    descripcion TEXT,
    estado VARCHAR(20) DEFAULT 'Programado' COMMENT 'Programado/En Proceso/Completado',
    vehiculo_id INT NOT NULL,
    cliente_id INT,  -- Puede ser NULL si el vehículo aún no se ha vendido
    mecanico_id INT NOT NULL,
    FOREIGN KEY (vehiculo_id) REFERENCES VEHICULO(vehiculo_id),
    FOREIGN KEY (cliente_id) REFERENCES CLIENTE(cliente_id),
    FOREIGN KEY (mecanico_id) REFERENCES MECANICO(mecanico_id)
);

-- Tabla de relación entre repuestos y mantenimientos
CREATE TABLE REPUESTO_MANTENIMIENTO (
    repuesto_mantenimiento_id INT AUTO_INCREMENT PRIMARY KEY,
    mantenimiento_id INT NOT NULL,
    repuesto_id INT NOT NULL,
    cantidad INT NOT NULL DEFAULT 1,
    precio_unitario DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (mantenimiento_id) REFERENCES MANTENIMIENTO(mantenimiento_id),
    FOREIGN KEY (repuesto_id) REFERENCES REPUESTO(repuesto_id)
);

-- Triggers para mantener la integridad de los datos

-- Actualizar el estado de disponibilidad del vehículo al venderse
DELIMITER //
CREATE TRIGGER after_detalle_venta_insert
AFTER INSERT ON DETALLE_VENTA
FOR EACH ROW
BEGIN
    UPDATE VEHICULO SET disponible = FALSE 
    WHERE vehiculo_id = NEW.vehiculo_id;
END//
DELIMITER ;

-- Verificar stock de repuestos al registrar un mantenimiento
DELIMITER //
CREATE TRIGGER before_repuesto_mantenimiento_insert
BEFORE INSERT ON REPUESTO_MANTENIMIENTO
FOR EACH ROW
BEGIN
    DECLARE stock_actual INT;
    
    SELECT cantidad_stock INTO stock_actual 
    FROM REPUESTO 
    WHERE repuesto_id = NEW.repuesto_id;
    
    IF stock_actual < NEW.cantidad THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'No hay suficiente stock del repuesto solicitado';
    END IF;
    
    -- Actualizar stock
    UPDATE REPUESTO 
    SET cantidad_stock = cantidad_stock - NEW.cantidad 
    WHERE repuesto_id = NEW.repuesto_id;
END//
DELIMITER ;

-- Vistas para facilitar consultas comunes

-- Vista de inventario de vehículos disponibles
CREATE VIEW vista_inventario_disponible AS
SELECT v.vehiculo_id, v.vin, v.marca, v.modelo, v.anio, v.precio, v.color, 
       c.nombre AS categoria, p.nombre AS proveedor
FROM VEHICULO v
JOIN CATEGORIA_VEHICULO c ON v.categoria_id = c.categoria_id
JOIN PROVEEDOR p ON v.proveedor_id = p.proveedor_id
WHERE v.disponible = TRUE;

-- Vista de ventas completas
CREATE VIEW vista_ventas_completas AS
SELECT v.venta_id, v.fecha_venta, v.total, 
       CONCAT(c.nombre, ' ', c.apellido) AS cliente,
       CONCAT(ve.nombre, ' ', ve.apellido) AS vendedor,
       s.nombre AS sucursal,
       COUNT(dv.detalle_venta_id) AS numero_vehiculos,
       CASE WHEN v.financiamiento_id IS NOT NULL THEN 'Financiada' ELSE 'Contado' END AS tipo_venta
FROM VENTA v
JOIN CLIENTE c ON v.cliente_id = c.cliente_id
JOIN VENDEDOR ve ON v.vendedor_id = ve.vendedor_id
JOIN SUCURSAL s ON ve.sucursal_id = s.sucursal_id
JOIN DETALLE_VENTA dv ON v.venta_id = dv.venta_id
GROUP BY v.venta_id;

-- Vista de historial de servicios por vehículo
CREATE VIEW vista_historial_servicios AS
SELECT v.vin, v.marca, v.modelo, 
       m.fecha_servicio, m.tipo_servicio, m.costo, m.estado,
       CONCAT(me.nombre, ' ', me.apellido) AS mecanico,
       CASE WHEN m.cliente_id IS NOT NULL 
            THEN CONCAT(c.nombre, ' ', c.apellido) 
            ELSE 'En inventario' END AS propietario
FROM MANTENIMIENTO m
JOIN VEHICULO v ON m.vehiculo_id = v.vehiculo_id
JOIN MECANICO me ON m.mecanico_id = me.mecanico_id
LEFT JOIN CLIENTE c ON m.cliente_id = c.cliente_id
ORDER BY v.vin, m.fecha_servicio DESC;

-- Procedimientos almacenados para operaciones comunes

-- Procedimiento para registrar una venta completa
DELIMITER //
CREATE PROCEDURE sp_registrar_venta(
    IN p_cliente_id INT,
    IN p_vendedor_id INT,
    IN p_vehiculo_id INT,
    IN p_metodo_pago VARCHAR(30),
    IN p_descuento DECIMAL(12,2),
    OUT p_venta_id INT
)
BEGIN
    DECLARE v_precio DECIMAL(12,2);
    DECLARE v_subtotal DECIMAL(12,2);
    DECLARE v_impuestos DECIMAL(12,2);
    DECLARE v_total DECIMAL(12,2);
    DECLARE v_disponible BOOLEAN;
    
    -- Verificar que el vehículo esté disponible
    SELECT precio, disponible INTO v_precio, v_disponible FROM VEHICULO WHERE vehiculo_id = p_vehiculo_id;
    
    IF NOT v_disponible THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El vehículo no está disponible para la venta';
    END IF;
    
    -- Calcular valores
    SET v_subtotal = v_precio - p_descuento;
    SET v_impuestos = v_subtotal * 0.16; -- Ejemplo con IVA del 16%
    SET v_total = v_subtotal + v_impuestos;
    
    -- Crear la venta
    INSERT INTO VENTA (fecha_venta, subtotal, impuestos, descuento, total, metodo_pago, cliente_id, vendedor_id)
    VALUES (CURRENT_DATE, v_subtotal, v_impuestos, p_descuento, v_total, p_metodo_pago, p_cliente_id, p_vendedor_id);
    
    SET p_venta_id = LAST_INSERT_ID();
    
    -- Crear el detalle de venta
    INSERT INTO DETALLE_VENTA (venta_id, vehiculo_id, precio_venta, descuento)
    VALUES (p_venta_id, p_vehiculo_id, v_precio, p_descuento);
    
    -- El trigger actualizará el estado del vehículo automáticamente
END//
DELIMITER ;

-- Procedimiento para programar un servicio de mantenimiento
DELIMITER //
CREATE PROCEDURE sp_programar_mantenimiento(
    IN p_vehiculo_id INT,
    IN p_cliente_id INT,
    IN p_mecanico_id INT,
    IN p_tipo_servicio VARCHAR(50),
    IN p_descripcion TEXT,
    IN p_costo DECIMAL(10,2)
)
BEGIN
    INSERT INTO MANTENIMIENTO (fecha_servicio, tipo_servicio, costo, descripcion, estado, vehiculo_id, cliente_id, mecanico_id)
    VALUES (CURRENT_DATE, p_tipo_servicio, p_costo, p_descripcion, 'Programado', p_vehiculo_id, p_cliente_id, p_mecanico_id);
    
    SELECT LAST_INSERT_ID() AS mantenimiento_id;
END//
DELIMITER ;

-- Índices para mejorar el rendimiento

-- Índice para búsquedas de vehículos por marca y modelo
CREATE INDEX idx_vehiculo_marca_modelo ON VEHICULO(marca, modelo);

-- Índice para búsquedas de clientes por nombre y apellido
CREATE INDEX idx_cliente_nombre_apellido ON CLIENTE(nombre, apellido);

-- Índice para búsquedas de ventas por fecha
CREATE INDEX idx_venta_fecha ON VENTA(fecha_venta);

-- Índice para búsquedas de mantenimientos por fecha y estado
CREATE INDEX idx_mantenimiento_fecha_estado ON MANTENIMIENTO(fecha_servicio, estado);
```
