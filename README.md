-- phpMyAdmin SQL Dump
-- version 5.1.3
-- https://www.phpmyadmin.net/
--
-- Servidor: 127.0.0.1
-- Tiempo de generación: 29-07-2022 a las 01:34:05
-- Versión del servidor: 10.4.24-MariaDB
-- Versión de PHP: 8.1.4

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";

--
-- Base de datos: `panaderia`
--

DELIMITER $$
--
-- Procedimientos
--
CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_actualizar` (IN `xnp` INT)   update detalle set id_ped=xnp where id_ped is null$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ActualizarDetalle` (IN `xiddet` INT, IN `xidpan` INT, IN `xcan` INT)   BEGIN
UPDATE detalle set can_dev = xcan where id_det= xiddet;
set @idcli = (select p.id_cliente from pedidos p inner join detalle d on p.id_ped = d.id_ped where d.id_det=xiddet);
set @can_pan = (select can_pan from detalle where id_det= xiddet);
set @tiempo = (SELECT TIMESTAMPDIFF(YEAR, fec_reg, CURDATE()) from clientes where id_cliente=@idcli);
if @tiempo>=4 THEN
DELETE from descuentos where id_cli = @idcli and id_pan=xidpan;
set @des = ((xcan/@can_pan)*3/0.2);
insert into descuentos (id_cli, id_pan, descuento, estado) values (@idcli, xidpan, @des,'A');
end if;
END$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_BuscarCliente` (IN `xcli` INT)   select * from clientes where id_cliente=xcli$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_BuscarDescuento` (IN `xidcli` INT, IN `xidpan` INT)   select descuento from descuentos where id_cli = xidcli and id_pan= xidpan$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_EliminarCliente` (IN `xid` INT)   DELETE
FROM
clientes WHERE id_cliente = xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_EliminarDetalle` (IN `xid` INT)   delete from detalle
where id_det = xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_EliminarFoto` (IN `xid` INT)   DELETE
FROM
fotos where id_foto = xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_EliminarPan` (IN `xid` INT)   delete 
from panes
where id_pan = xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_EliminarPedido` (IN `xid` INT)   BEGIN
delete FROM pedidos where id_ped = xid;
END$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_EliminarRecojo` (IN `xid` INT)   DELETE
from recojo where id_recojo=xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_EliminarUsuario` (IN `xid` INT)   DELETE
FROM usuarios where id_usuario=xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ingresarCliente` (IN `xtipo` CHAR(1), IN `xnro` VARCHAR(11), IN `xnom` VARCHAR(35), IN `xcorreo` VARCHAR(35), IN `xdir` VARCHAR(35), IN `xdis` INT(1), IN `xusu` VARCHAR(20), IN `xcon` VARCHAR(20), IN `xfec` VARCHAR(12), IN `xest` CHAR(1))   INSERT into clientes (tipo_cliente,nro_doc, nombre, correo, direccion, id_dis, nom_usuario,con_usuario, fec_reg, estado)
values (xtipo, xnro, xnom, xcorreo, xdir, xdis, xusu, xcon, xfec, xest)$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_IngresarDetalle` (IN `xidpan` INT, IN `xcan` INT, IN `xdescto` DECIMAL(7,2))   Begin
set @xpre= (select pre_pan from panes WHERE id_pan = xidpan);

set @ximp = (xcan * @xpre - (xcan * @xpre * xdescto/100));

INSERT into detalle (id_pan, can_pan, descto, imp_pan) values (xidpan, xcan, xdescto, @ximp);
END$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_IngresarFoto` (IN `xdes` VARCHAR(35), IN `xidcli` INT, IN `ximg` VARCHAR(40), IN `xfec` VARCHAR(12), IN `xest` CHAR(1))   insert into fotos (des_foto, id_cliente, img_foto, fec_foto, estado) values (xdes, xidcli, ximg, xfec, xest )$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_IngresarPan` (IN `xnom` VARCHAR(35), IN `xdes` VARCHAR(40), IN `xpre` DECIMAL(7,2), IN `ximg` VARCHAR(45), IN `xest` CHAR(1))   insert into panes (nom_pan, des_pan, pre_pan, img_pan, estado )values (xnom, xdes, xpre, ximg, 'A')$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_IngresarPedido` (IN `xidcli` INT, IN `xfec` DATE, IN `xtot` DECIMAL(7,2), IN `xest` CHAR(1))   BEGIN
set @total = (select sum(imp_pan) from detalle where id_ped is null);
INSERT INTO pedidos (id_cliente, fec_ped, tot_ped, estado) values (xidcli, xfec, @total, xest);
select MAX(id_ped) as id from pedidos;
END$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_IngresarRecojo` (IN `xidped` INT)   BEGIN
set @panes = (select sum(can_dev) from pedidos p inner join detalle d on p.id_ped=d.id_ped where p.id_ped=xidped);
set @importe = (select sum(can_dev*imp_pan/can_pan) from detalle where id_ped=xidped);
insert into recojo (id_ped, fec_recojo, tot_panes, tot_recojo, estado) values (xidped,CURDATE(),@panes,@importe ,'A');
update pedidos set estado='R' where id_ped=xidped;
END$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_IngresarUsuario` (IN `xdni` CHAR(8), IN `xnom` VARCHAR(35), IN `xcorreo` VARCHAR(45), IN `xusu` VARCHAR(20), IN `xcon` VARCHAR(20), IN `xidrol` INT, IN `xfec` DATE)   INSERT INTO usuarios(nro_doc, nombre, correo, nom_usuario, con_usuario, id_rol, fec_reg, estado) values (xdni, xnom, xcorreo, xusu, xcon, xidrol, xfec,'A')$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaClientes` ()   SELECT c.id_cliente, c.tipo_cliente,c.nro_doc,c.nombre, c.correo, c.direccion, d.nom_dis, c.fec_reg,c.estado
from clientes c inner join distritos d on c.id_dis=d.id_dis$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaDetalles` ()   SELECT d.id_det, p.nom_pan, d.can_pan, d.descto, d.imp_pan
from detalle d inner join panes p on d.id_pan=p.id_pan
WHERE id_ped is null$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaFotos` ()   SELECT f.id_foto, f.des_foto, c.nombre, f.fec_foto, f.estado
from fotos f inner join clientes c
on f.id_cliente=c.id_cliente$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaPanes` ()   select p.id_pan, p.nom_pan,p.des_pan, p.pre_pan, p.img_pan, p.estado
from panes p$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaPedidoDetalles` (IN `xidped` INT)   SELECT d.id_det,p.id_pan, p.nom_pan, d.can_pan, d.descto, d.imp_pan
from detalle d inner join panes p on d.id_pan=p.id_pan
WHERE id_ped =xidped$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaPedidos` ()   SELECT p.id_ped, p.fec_ped, c.nombre, p.tot_ped, p.estado
FROM pedidos p inner join clientes c on c.id_cliente=p.id_cliente ORDER by p.id_ped DESC$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaPedidosA` ()   SELECT p.id_ped, p.fec_ped, c.nombre, p.tot_ped, p.estado
FROM pedidos p inner join clientes c on c.id_cliente=p.id_cliente where p.estado='A' order by 1 DESC$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaRecojos` ()   SELECT r.id_recojo, r.fec_recojo, p.id_ped, c.nombre, r.tot_panes, r.tot_recojo, r.estado
FROM
recojo r inner join pedidos p on r.id_ped=p.id_ped
INNER JOIN clientes c on p.id_cliente=c.id_cliente$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaRoles` ()   SELECT id_rol, nom_rol 
FROM rol$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ListaUsuarios` ()  NO SQL SELECT u.id_usuario, u.nom_usuario, r.nom_rol, u.nombre, u.estado
FROM usuarios u inner join rol r on u.id_rol=r.id_rol
ORDER by u.nom_usuario$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ModificaPan` (IN `xid` INT, IN `xnom` VARCHAR(35), IN `xdes` VARCHAR(45), IN `xpre` DECIMAL(7,2), IN `ximg` VARCHAR(45), IN `xest` CHAR(1))   update panes set nom_pan=xnom, des_pan=xdes, pre_pan=xpre, img_pan= ximg, estado=xest 
where id_pan = xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_ModificarUsuario` (IN `xid` INT, IN `xdni` CHAR(8), IN `xnom` VARCHAR(35), IN `xcorreo` VARCHAR(45), IN `xusu` VARCHAR(20), IN `xcon` VARCHAR(20), IN `xidrol` INT, IN `xfec` DATE, IN `xest` CHAR(1))   update usuarios set nro_doc=xdni, nombre=xnom, correo=xcorreo, nom_usuario= xusu, con_usuario= xcon, id_rol=xidrol, fec_reg=xfec, estado= xest
where id_usuario=xid$$

CREATE DEFINER=`root`@`localhost` PROCEDURE `usp_validarLogin` (IN `usu` VARCHAR(20), IN `con` VARCHAR(20))  NO SQL SELECT id_usuario, nom_usuario, con_usuario, id_rol
from usuarios
where nom_usuario=usu and con_usuario = con$$

DELIMITER ;

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `clientes`
--

CREATE TABLE `clientes` (
  `id_cliente` int(11) NOT NULL,
  `tipo_cliente` char(1) COLLATE utf8_spanish2_ci NOT NULL,
  `nro_doc` varchar(11) COLLATE utf8_spanish2_ci NOT NULL,
  `nombre` varchar(35) COLLATE utf8_spanish2_ci NOT NULL,
  `correo` varchar(35) COLLATE utf8_spanish2_ci NOT NULL,
  `direccion` varchar(40) COLLATE utf8_spanish2_ci NOT NULL,
  `id_dis` int(11) NOT NULL,
  `nom_usuario` varchar(20) COLLATE utf8_spanish2_ci NOT NULL,
  `con_usuario` varchar(20) COLLATE utf8_spanish2_ci NOT NULL,
  `fec_reg` date NOT NULL,
  `descto` decimal(7,2) NOT NULL,
  `estado` char(1) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `clientes`
--

INSERT INTO `clientes` (`id_cliente`, `tipo_cliente`, `nro_doc`, `nombre`, `correo`, `direccion`, `id_dis`, `nom_usuario`, `con_usuario`, `fec_reg`, `descto`, `estado`) VALUES
(1, '1', '12345678901', 'Tienda Jaimitos', 'fabiola@gmail.com', 'Av. Los alamos 780', 1, 'rhuarcaya', '123', '2020-07-17', '0.00', 'A'),
(2, '1', '7889635214', 'Tienda El Chino', 'chino@yahoo.es', 'Av. Sucre 780 - Los Olivos', 1, 'chino', '1234', '2015-07-17', '0.00', 'A'),
(3, '1', '5555', 'MiniMarket Faby', 'f', 'Jr. San Pablo 784 - Ind', 1, 'rhuarcaya', '123', '2022-07-17', '0.00', 'A'),
(4, '1', '89876545676', 'Tienda Dianas', 'dianas@gmail.com', 'Jr. Loli 890 - Independencia', 1, 'floli', '123', '2010-07-17', '0.00', 'A'),
(5, '1', '09098767905', 'Miguel Angel', 'ma@gmail.com', 'jr arequpia 450', 1, 'mangel', '123', '2020-07-18', '0.00', 'A');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `descuentos`
--

CREATE TABLE `descuentos` (
  `id_des` int(11) NOT NULL,
  `id_cli` int(11) NOT NULL,
  `id_pan` int(11) NOT NULL,
  `descuento` decimal(7,2) NOT NULL,
  `estado` char(1) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `descuentos`
--

INSERT INTO `descuentos` (`id_des`, `id_cli`, `id_pan`, `descuento`, `estado`) VALUES
(21, 4, 4, '0.52', 'A'),
(22, 2, 3, '1.20', 'A'),
(26, 2, 6, '4.50', 'A'),
(27, 2, 5, '14.55', 'A'),
(28, 4, 6, '3.75', 'A');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `detalle`
--

CREATE TABLE `detalle` (
  `id_det` int(11) NOT NULL,
  `id_ped` int(11) DEFAULT NULL,
  `id_pan` int(11) NOT NULL,
  `can_pan` int(11) NOT NULL,
  `descto` decimal(7,2) DEFAULT NULL,
  `can_dev` int(11) DEFAULT NULL,
  `imp_pan` decimal(7,2) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `detalle`
--

INSERT INTO `detalle` (`id_det`, `id_ped`, `id_pan`, `can_pan`, `descto`, `can_dev`, `imp_pan`) VALUES
(5, 80, 2, 100, '0.00', 5, '70.00'),
(6, 81, 5, 150, '0.00', 50, '150.00'),
(7, 82, 3, 180, '0.00', NULL, '144.00'),
(8, 83, 3, 100, '0.00', 20, '80.00'),
(9, 84, 2, 100, '0.00', 20, '70.00'),
(10, 85, 4, 150, '0.00', 13, '135.00'),
(18, 86, 3, 100, '0.00', 20, '80.00'),
(19, 87, 3, 1, '1.20', NULL, '0.79'),
(20, 88, 5, 100, '0.00', 20, '100.00'),
(21, 89, 5, 100, '3.00', 97, '97.00'),
(22, 89, 6, 10, '0.00', 3, '2.50'),
(23, 90, 6, 100, '0.00', 25, '25.00'),
(24, 91, 2, 100, '0.00', NULL, '70.00'),
(25, 91, 6, 50, '3.75', NULL, '12.03'),
(26, 91, 2, 1, '0.00', NULL, '0.70'),
(27, 92, 2, 100, '0.00', NULL, '70.00'),
(28, 92, 4, 100, '0.00', NULL, '90.00');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `distritos`
--

CREATE TABLE `distritos` (
  `id_dis` int(11) NOT NULL,
  `nom_dis` varchar(25) COLLATE utf8_spanish2_ci NOT NULL,
  `cod_postal` char(3) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `distritos`
--

INSERT INTO `distritos` (`id_dis`, `nom_dis`, `cod_postal`) VALUES
(1, 'Cercado de Lima', '001'),
(2, 'La Victoria', '002'),
(3, 'Jesús María', '003'),
(4, 'Lince', '004'),
(5, 'Pueblo Libre', '006'),
(6, 'Surquillo', '007'),
(7, 'Miraflores', '010'),
(8, 'La Molina', '011');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `fotos`
--

CREATE TABLE `fotos` (
  `id_foto` int(11) NOT NULL,
  `des_foto` varchar(30) COLLATE utf8_spanish2_ci NOT NULL,
  `id_cliente` int(11) NOT NULL,
  `img_foto` varchar(50) COLLATE utf8_spanish2_ci NOT NULL,
  `fec_foto` date NOT NULL,
  `estado` char(1) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `fotos`
--

INSERT INTO `fotos` (`id_foto`, `des_foto`, `id_cliente`, `img_foto`, `fec_foto`, `estado`) VALUES
(1, 'Productos bien ubicados', 1, 'cesta_pan.jpg', '2022-07-01', 'A'),
(2, 'Falta diseño de  soporte', 1, 'aws-academy-educator.png', '2022-07-18', 'A'),
(11, 'mostrador 15', 5, 'cesta_pan.jpg', '2022-07-15', 'A'),
(12, 'Mostrador de la izquierda', 4, 'canastilla_pan.jpg', '2022-07-20', 'A');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `panes`
--

CREATE TABLE `panes` (
  `id_pan` int(11) NOT NULL,
  `nom_pan` varchar(35) COLLATE utf8_spanish2_ci NOT NULL,
  `des_pan` varchar(100) COLLATE utf8_spanish2_ci NOT NULL,
  `pre_pan` decimal(7,2) NOT NULL,
  `img_pan` varchar(45) COLLATE utf8_spanish2_ci NOT NULL,
  `estado` varchar(1) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `panes`
--

INSERT INTO `panes` (`id_pan`, `nom_pan`, `des_pan`, `pre_pan`, `img_pan`, `estado`) VALUES
(1, 'Pan de trigo', 'Contiene trigo, levadura 3%, sal entre otros ', '0.50', 'canastilla_pan.jpg', 'A'),
(2, 'Pan de maiz', 'Contiene maiz importado con levadura importad', '0.70', 'cesta_pan02.jpg', 'A'),
(3, 'Pan de centeno', 'contiene ...', '0.80', 'cesta_pan.jpg', 'A'),
(4, 'Pan de cebada', 'Fabricado con cebada nacional', '0.90', 'cesta.jpg', 'A'),
(5, 'Pan de wong', 'Pan con semilla importada y frutillas', '1.00', 'cesta_pan02.jpg', 'A'),
(6, 'Pan caracol', 'Pan dulce', '0.25', 'cerebro.png', 'A');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `pedidos`
--

CREATE TABLE `pedidos` (
  `id_ped` int(11) NOT NULL,
  `id_cliente` int(11) NOT NULL,
  `fec_ped` date NOT NULL,
  `tot_ped` decimal(7,2) NOT NULL,
  `estado` char(1) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `pedidos`
--

INSERT INTO `pedidos` (`id_ped`, `id_cliente`, `fec_ped`, `tot_ped`, `estado`) VALUES
(81, 5, '2022-07-28', '150.00', 'R'),
(83, 5, '2022-07-28', '80.00', 'R'),
(84, 1, '2022-07-28', '70.00', 'R'),
(85, 4, '2022-07-28', '135.00', 'R'),
(86, 2, '2022-07-28', '80.00', 'R'),
(88, 2, '2022-07-28', '100.00', 'R'),
(89, 2, '2022-07-28', '99.50', 'A'),
(90, 4, '2022-07-29', '25.00', 'R'),
(91, 4, '2022-07-29', '82.73', 'A'),
(92, 1, '2022-07-29', '160.00', 'A');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `recojo`
--

CREATE TABLE `recojo` (
  `id_recojo` int(11) NOT NULL,
  `fec_recojo` date NOT NULL,
  `id_ped` int(11) NOT NULL,
  `tot_panes` int(11) DEFAULT NULL,
  `tot_recojo` decimal(7,2) NOT NULL,
  `estado` char(1) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `recojo`
--

INSERT INTO `recojo` (`id_recojo`, `fec_recojo`, `id_ped`, `tot_panes`, `tot_recojo`, `estado`) VALUES
(10, '2022-07-27', 81, 50, '50.00', 'A'),
(12, '2022-07-27', 83, 20, '16.00', 'A'),
(13, '2022-07-27', 84, 20, '14.00', 'A'),
(14, '2022-07-27', 85, 13, '11.70', 'A'),
(15, '2022-07-27', 86, 20, '16.00', 'A'),
(16, '2022-07-27', 88, 20, '20.00', 'A'),
(17, '2022-07-28', 90, 25, '6.25', 'A');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `rol`
--

CREATE TABLE `rol` (
  `id_rol` int(11) NOT NULL,
  `nom_rol` varchar(20) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `rol`
--

INSERT INTO `rol` (`id_rol`, `nom_rol`) VALUES
(1, 'Administrador'),
(2, 'Supervisor'),
(3, 'Operador'),
(4, 'Subir fotos'),
(5, 'Público');

-- --------------------------------------------------------

--
-- Estructura de tabla para la tabla `usuarios`
--

CREATE TABLE `usuarios` (
  `id_usuario` int(11) NOT NULL,
  `nro_doc` varchar(11) COLLATE utf8_spanish2_ci NOT NULL,
  `nombre` varchar(35) COLLATE utf8_spanish2_ci NOT NULL,
  `correo` varchar(35) COLLATE utf8_spanish2_ci NOT NULL,
  `nom_usuario` varchar(20) COLLATE utf8_spanish2_ci NOT NULL,
  `con_usuario` varchar(20) COLLATE utf8_spanish2_ci NOT NULL,
  `id_rol` int(11) NOT NULL,
  `fec_reg` date NOT NULL,
  `estado` char(1) COLLATE utf8_spanish2_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish2_ci;

--
-- Volcado de datos para la tabla `usuarios`
--

INSERT INTO `usuarios` (`id_usuario`, `nro_doc`, `nombre`, `correo`, `nom_usuario`, `con_usuario`, `id_rol`, `fec_reg`, `estado`) VALUES
(1, '78945621', 'Juan', '', 'jguillen', '123', 1, '2021-06-25', 'A'),
(2, '98745687', 'Admin', '', 'admin', 'admin', 1, '2021-06-25', 'A'),
(3, '58964578', 'Operador', '', 'juan', '123', 1, '2021-06-02', 'A'),
(4, '12345678', 'Fabiola Huarcaya Troncos', 'fehuarcaya@hotmail.com', 'fhuarcaya', '123', 4, '2022-07-21', 'A'),
(38, '1213123', 'isaac', 'qweqwe', 'isaac', '1234', 4, '2022-07-25', 'A');

--
-- Índices para tablas volcadas
--

--
-- Indices de la tabla `clientes`
--
ALTER TABLE `clientes`
  ADD PRIMARY KEY (`id_cliente`),
  ADD KEY `id_dis` (`id_dis`);

--
-- Indices de la tabla `descuentos`
--
ALTER TABLE `descuentos`
  ADD PRIMARY KEY (`id_des`),
  ADD KEY `id_cli` (`id_cli`),
  ADD KEY `id_pan` (`id_pan`);

--
-- Indices de la tabla `detalle`
--
ALTER TABLE `detalle`
  ADD PRIMARY KEY (`id_det`);

--
-- Indices de la tabla `distritos`
--
ALTER TABLE `distritos`
  ADD PRIMARY KEY (`id_dis`);

--
-- Indices de la tabla `fotos`
--
ALTER TABLE `fotos`
  ADD PRIMARY KEY (`id_foto`),
  ADD KEY `id_cliente` (`id_cliente`);

--
-- Indices de la tabla `panes`
--
ALTER TABLE `panes`
  ADD PRIMARY KEY (`id_pan`);

--
-- Indices de la tabla `pedidos`
--
ALTER TABLE `pedidos`
  ADD PRIMARY KEY (`id_ped`),
  ADD KEY `id_cliente` (`id_cliente`);

--
-- Indices de la tabla `recojo`
--
ALTER TABLE `recojo`
  ADD PRIMARY KEY (`id_recojo`),
  ADD KEY `id_ped` (`id_ped`);

--
-- Indices de la tabla `rol`
--
ALTER TABLE `rol`
  ADD PRIMARY KEY (`id_rol`);

--
-- Indices de la tabla `usuarios`
--
ALTER TABLE `usuarios`
  ADD PRIMARY KEY (`id_usuario`),
  ADD KEY `id_rol` (`id_rol`);

--
-- AUTO_INCREMENT de las tablas volcadas
--

--
-- AUTO_INCREMENT de la tabla `clientes`
--
ALTER TABLE `clientes`
  MODIFY `id_cliente` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=8;

--
-- AUTO_INCREMENT de la tabla `descuentos`
--
ALTER TABLE `descuentos`
  MODIFY `id_des` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=29;

--
-- AUTO_INCREMENT de la tabla `detalle`
--
ALTER TABLE `detalle`
  MODIFY `id_det` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=29;

--
-- AUTO_INCREMENT de la tabla `distritos`
--
ALTER TABLE `distritos`
  MODIFY `id_dis` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=9;

--
-- AUTO_INCREMENT de la tabla `fotos`
--
ALTER TABLE `fotos`
  MODIFY `id_foto` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=13;

--
-- AUTO_INCREMENT de la tabla `panes`
--
ALTER TABLE `panes`
  MODIFY `id_pan` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=9;

--
-- AUTO_INCREMENT de la tabla `pedidos`
--
ALTER TABLE `pedidos`
  MODIFY `id_ped` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=93;

--
-- AUTO_INCREMENT de la tabla `recojo`
--
ALTER TABLE `recojo`
  MODIFY `id_recojo` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=18;

--
-- AUTO_INCREMENT de la tabla `rol`
--
ALTER TABLE `rol`
  MODIFY `id_rol` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=6;

--
-- AUTO_INCREMENT de la tabla `usuarios`
--
ALTER TABLE `usuarios`
  MODIFY `id_usuario` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=39;

--
-- Restricciones para tablas volcadas
--

--
-- Filtros para la tabla `clientes`
--
ALTER TABLE `clientes`
  ADD CONSTRAINT `clientes_ibfk_1` FOREIGN KEY (`id_dis`) REFERENCES `distritos` (`id_dis`);

--
-- Filtros para la tabla `descuentos`
--
ALTER TABLE `descuentos`
  ADD CONSTRAINT `descuentos_ibfk_1` FOREIGN KEY (`id_cli`) REFERENCES `clientes` (`id_cliente`),
  ADD CONSTRAINT `descuentos_ibfk_2` FOREIGN KEY (`id_pan`) REFERENCES `panes` (`id_pan`);

--
-- Filtros para la tabla `fotos`
--
ALTER TABLE `fotos`
  ADD CONSTRAINT `fotos_ibfk_1` FOREIGN KEY (`id_cliente`) REFERENCES `clientes` (`id_cliente`);

--
-- Filtros para la tabla `pedidos`
--
ALTER TABLE `pedidos`
  ADD CONSTRAINT `pedidos_ibfk_1` FOREIGN KEY (`id_cliente`) REFERENCES `clientes` (`id_cliente`);

--
-- Filtros para la tabla `recojo`
--
ALTER TABLE `recojo`
  ADD CONSTRAINT `recojo_ibfk_1` FOREIGN KEY (`id_ped`) REFERENCES `pedidos` (`id_ped`);

--
-- Filtros para la tabla `usuarios`
--
ALTER TABLE `usuarios`
  ADD CONSTRAINT `usuarios_ibfk_1` FOREIGN KEY (`id_rol`) REFERENCES `rol` (`id_rol`);
COMMIT;
