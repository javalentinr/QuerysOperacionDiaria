#Clientes sin unidad
#No hay

SELECT * FROM db_besta_management_prod.dat_cliente dc
WHERE dc.nIdStatusCliente IN (2,3,4,9) /* Arrendamiento, prorroga, plan de pago, valor residual */
	AND dc.bActivo = 1
	AND dc.nIdActivo IS NULL;

-- Query de ordenes que quieren estar pagadas pero no sale factura
#No hay

SELECT * FROM db_besta_management_prod.ops_orden_pago WHERE `bEnviadoFact` = 1 AND `bActivo` = 1  AND `nIdStatusOP` ORDER BY `nIdStatusOP` DESC;


  -- Facturas sin relaciÓn de OP arr. plan de pago, VR,  de Gastos notariales, pagare automatico
  #hay
  #131906 Es de chocolate 
  
  SELECT
  df.*,
  oop.nIdOpsOP,
  odgn.nIdOpsGN,
  dpad.nIdPagareAutomaticDetalle
  
  FROM db_besta_management_prod.dat_facturas df
  
  LEFT JOIN db_besta_management_prod.ops_orden_pago oop
  ON oop.nIdFactura = df.nIdFactura
  AND oop.bActivo = 1
  
  LEFT JOIN db_besta_management_prod.ops_deposito_gastos_notariales odgn
  ON odgn.nIdFactura = df.nIdFactura
  AND odgn.bActivo = 1
  
  LEFT JOIN db_besta_management_prod.dat_pagare_automatic_detalle AS dpad
  ON dpad.nIdFactura = df.nIdFactura
  AND dpad.bActivo = 1
  
  WHERE
  df.bActivo = 1
  AND df.nIdStatusFactura NOT IN (3,4)
  AND oop.nIdOpsOP IS NULL
  AND odgn.nIdOpsGN IS NULL
  AND dpad.nIdFactura IS NULL
  AND YEAR(df.Date) = 2026
  AND df.nIdCliente NOT IN (1381) -- facturas son las OP que pidieron ayer que se inactivara del JESUS MARIA LOPEZ MAZZO, el cliente de Salvador que cayeron y le dieron otro carro, por eso no aparecen relacionadas
  ;




/*FACTURAS CANCELADAS CON OP PAGADA*/
#Hay 6 Son pasadas

SELECT
df.*,
oop.nIdOpsOP,
odgn.nIdOpsGN,
dpad.nIdPagareAutomaticDetalle
FROM db_besta_management_prod.dat_facturas df
LEFT JOIN db_besta_management_prod.ops_orden_pago oop
ON oop.nIdFactura = df.nIdFactura
AND oop.bActivo = 1
LEFT JOIN db_besta_management_prod.ops_deposito_gastos_notariales odgn
ON odgn.nIdFactura = df.nIdFactura
AND odgn.bActivo = 1
LEFT JOIN db_besta_management_prod.dat_pagare_automatic_detalle AS dpad
ON dpad.nIdFactura = df.nIdFactura
AND dpad.bActivo = 1
WHERE
df.bActivo = 1
AND df.nIdStatusFactura IN (3)
AND (oop.nIdStatusOp = 2 OR odgn.nIdStatusOp = 2 OR dpad.nIdStatusOp = 2  )
AND YEAR(df.Date) >= 2026
AND df.nIdCliente NOT IN (1381) 
;




-- Consulta si existen complementos duplicados. AVISARLE A ALDO
#No hay

SELECT
    COUNT(*) AS nTotalComplemento,
    nIdFactura,
    `dFecRegistro`
FROM
    db_besta_management_prod.dat_complemento_pagos
WHERE
    bActivo = 1
AND nIdStatusFactura != 3
GROUP BY
    `nIdFactura`
HAVING
    nTotalComplemento > 1 AND YEAR(dFecRegistro) = 2025
ORDER BY
    `dat_complemento_pagos`.`dFecRegistro` ASC;






  -- Obtiene las OP sin pagar con factura pagada
  #Es la del 2023 
  
  SELECT
    df.nIdFactura,
    oop.nIdOpsOP,    
    oop.nIdPagoParcial,
    oop.nIdArrendamiento,
    oop.nIdProrroga,
    oop.nIdPlanPago,
    oop.nIdValorResidual,    
    oop.nIdFactura,
        IFNULL(COALESCE(
        dpagop.sReferenciaPaycash,
        cop.sReferenciaPaycash,
        dp.sReferenciaPaycash,
        dplanp.sReferenciaPaycash,
        cvr.sReferenciaPaycash
        ),'0') AS sReferenciaPaycash ,        
        drpo.nIdOpPago,
        (   SELECT  dpp.sReference FROM db_besta_management_prod.dat_pagos_paycash dpp
            WHERE   dpp.nIdPagoPaycash = drpo.nIdPagoPaycash
            AND dpp.bActivo = 1 LIMIT 1  ) AS sReferenciaPaycashPago,
        (   SELECT  dpp.dDate FROM db_besta_management_prod.dat_pagos_paycash dpp
            WHERE   dpp.nIdPagoPaycash = drpo.nIdPagoPaycash
            AND dpp.bActivo = 1 LIMIT 1  ) AS dDatePago,
        COUNT(dcp.nIdComplentoPago) AS nTotalComplemento,
        oop.nIdStatusOp,
        df.nIdStatusFactura
  
  FROM db_besta_management_prod.`dat_facturas` df

        JOIN db_besta_management_prod.ops_orden_pago oop
        ON oop.nIdFactura = df.nIdFactura
        AND oop.bActivo = 1
  
        LEFT JOIN db_besta_management_prod.dat_complemento_pagos dcp
        ON dcp.nIdFactura = df.nIdFactura

            LEFT JOIN db_besta_management_prod.dat_pago_parcial dpagop
            ON dpagop.nIdPagoParcial = oop.nIdPagoParcial
      
            LEFT JOIN db_besta_management_prod.cfg_orden_pago cop
            ON cop.nIdOrdenPago = oop.nIdArrendamiento      
            AND cop.bActivo = 1
      
            LEFT JOIN db_besta_management_prod.dat_prorroga dp
            ON dp.nIdProrroga = oop.nIdProrroga
      
            LEFT JOIN db_besta_management_prod.dat_plan_pago dplanp
            ON dplanp.nIdPlanPago = oop.nIdPlanPago
      
            LEFT JOIN db_besta_management_prod.cfg_valor_residual cvr
            ON cvr.nIdValorResidual = oop.nIdValorResidual
            AND cvr.bActivo = 1
                      
            LEFT JOIN db_besta_management_prod.dat_relacion_pagos_op drpo
            ON drpo.nIdOrdenPago = oop.nIdOpsOP
            AND drpo.bActivo = 1
      
  WHERE
    df.bActivo = 1    
    AND df.nIdStatusFactura = 2 /*Pagada*/
    AND oop.nIdStatusOp != 2 /*Sin pagar*/
  GROUP BY df.nIdFactura ;





-- Busca si una OP tiene mas de un pago relacionado.
#No hay

SELECT
  COUNT(`nIdOrdenPago`) AS nTotal,
  nIdOrdenPago,
  GROUP_CONCAT(`nIdOpPago`) AS nIdsOpPago,
  `nIdPagoPaycash`
FROM
  db_besta_management_prod.dat_relacion_pagos_op
WHERE
  `bActivo` = 1
GROUP BY
  `nIdOrdenPago`
HAVING
  nTotal > 1;





-- CONSULTA QUE OBTIENE LAS OP QUE ESTAN VENCIDAS SIN FACTURAR DE CLIENTES QUE NO ESTAN CAIDOS, EJECUTAR MARTES DESPUES DE LAS 12:00pm y Miércoles en la mañana.

SELECT
    IF(dc.bFacturaPublicoGeneral = 1,'PUBLICO EN GENERAL','DATOS FISCALES CLIENTE')AS sTipoFacturacion,
    oop.*
FROM
    db_besta_management_prod.ops_orden_pago oop
    JOIN db_besta_management_prod.dat_cliente dc
    ON dc.nIdCliente = oop.nIdCliente
WHERE
    oop.nIdStatusOp = 4 AND oop.nIdFactura = 0 AND oop.nIdCliente NOT IN
    ( SELECT
        nIdCliente
    FROM
        db_besta_management_prod.dat_cliente
    WHERE
        nIdStatusCliente = 7 AND bActivo = 1
    )
	 AND oop.bActivo = 1 ;





-- Facturas de gastos notariales sin generar
#Hay

SELECT
    odgn.nIdOpsGN,
    odgn.nIdCliente,
    CONCAT(dc.sNombre,' ',dc.sApellidoP,' ',dc.sApellidoM) AS  sNombreCliente   ,
    odgn.sReferenciaPaycash,
    SUBSTRING(odgn.dFecRegistro,1,10) AS dFecha,
    SUBSTRING(odgn.dFecRegistro,12,9) AS dHora,
    odgn.sObervaciones
FROM
    db_besta_management_prod.`ops_deposito_gastos_notariales` odgn
    JOIN db_besta_management_prod.dat_cliente dc
    ON dc.nIdCliente = odgn.nIdCliente
    AND dc.bActivo = 1  
WHERE
    odgn.`nIdFactura` IS NULL 
	 AND odgn.`nIdStatusOp` = 2
	 AND odgn.bActivo = 1
ORDER BY
    odgn.`nIdFactura` ASC, dFecha DESC;





-- Gastos notariales sin relacion de pago (Revision de factura)
#Hay

SELECT
  odgn.*,drpogn.nIdOpPagoGN
FROM
  db_besta_management_prod.ops_deposito_gastos_notariales odgn
LEFT JOIN db_besta_management_prod.dat_relacion_pagos_op_gastos_notariales drpogn
ON drpogn.nIdOpsGN = odgn.nIdOpsGN
AND drpogn.bActivo = 1
WHERE
  odgn.nIdStatusOp = 2 -- Pagado
  AND odgn.bActivo = 1
  AND drpogn.nIdOpPagoGN IS NULL
ORDER BY 
	odgn.dFecRegistro DESC	
;





-- Consulta para obtener usuarios sin idcliente
#No hay

SELECT * FROM db_besta_management_prod.dat_usuarios WHERE `nIdTipoUsuario` = 2 AND `nIdCliente` IS NULL;




-- Verifica si hay pagos que no se han conciliado
call sp_select_tarea_conciliacion_pago_pagare_automatic();
-- Verifica si hay clientes que deberian caerse segun las reglas
call sp_select_tarea_proceso_caido_pagare_automatic();
-- Verifica deuda de clientes pagare automatico
CALL sp_select_info_deuda_pagare_automatic(1);






-- Consulta que obtiene las OPs de VR que no se han conciliado  Avisar a Aldo
#No hay

SELECT
dpp.*
FROM db_besta_management_prod.dat_pagos_paycash dpp
    LEFT JOIN db_besta_management_prod.dat_relacion_pagos_op dro
    ON dro.nIdPagoPaycash = dpp.nIdPagoPaycash
    AND dro.bActivo = 1
   
WHERE dpp.bActivo = 1
AND dro.nIdOpPago IS NULL
AND dpp.sReference IN (SELECT sReferenciaPaycash FROM `cfg_valor_residual` WHERE  bActivo = 1 AND dFecVencimiento >= '2025-01-01' )
;





-- Consulta para buscar un texto dentro de PS´S y Disparadores:
#No hay

SELECT  ROUTINE_NAME
FROM information_schema.routines
WHERE routine_definition LIKE '%hst_dat_activo_cliente%'
ORDER BY routine_name;






-- Búsqueda en 'dat_activo' de registros con sVin que incluye 'xxx'
#No hay

SELECT *
FROM `dat_activo`
WHERE `sVin`
LIKE '%xxx%'






-- CONSULTA QUE OBTIENE EL DETALLE DE LOS VALOR RESIDUALES PARA CAMBIO DE STATUS DEL CLIENTE

SELECT
    cvr.nIdValorResidual,
    csvr.sNombre AS sIdStatusValorResidual,
    CONCAT_WS(' ', IFNULL(dc.sNombre, ''), IFNULL(dc.sApellidoP, ''), IFNULL(dc.sApellidoM, '')) AS sNombreCliente,
    csc.sNombre AS sIdStatusCliente,
    DATE_FORMAT( cvr.dFechaInicio , '%Y-%m-%d') AS dFechaInicio,
    DATE_FORMAT( cvr.dFecVencimiento , '%Y-%m-%d')  AS dFecVencimiento,
    (SELECT COUNT(nIdValorResidual) FROM db_besta_management_prod.cfg_valor_residual WHERE nIdCliente = cvr.nIdCliente AND bActivo = 1 ) AS nTotalVR,
    cvr.nTotal,
    (SELECT
        COUNT(oop.nIdOpsOP) FROM db_besta_management_prod.ops_orden_pago oop
        WHERE oop.bActivo = 1
        AND oop.nIdStatusOp = 2
        AND oop.nIdValorResidual = cvr.nIdValorResidual
        AND oop.nTotal = cvr.nTotal
    ) AS nTotalOPPagadas,
    (SELECT
        COUNT(oop.nIdOpsOP) FROM db_besta_management_prod.ops_orden_pago oop
        WHERE oop.bActivo = 1
        AND oop.nIdStatusOp IN (1,4)
        AND oop.nIdValorResidual = cvr.nIdValorResidual
        AND oop.nTotal = cvr.nTotal
    ) AS nTotalOPPendientesPago
   
FROM  db_besta_management_prod.cfg_valor_residual cvr
JOIN  db_besta_management_prod.cat_status_valor_residual csvr
    ON csvr.nIdStatusValorResidual = cvr.nIdStatusValorResidual
    AND csvr.bActivo = 1
JOIN db_besta_management_prod.dat_cliente AS dc
    ON dc.nIdCliente = cvr.nIdCliente
    AND dc.bActivo = 1
JOIN db_besta_management_prod.cat_status_cliente csc
    ON csc.nIdStatusCliente = dc.nIdStatusCliente
   
WHERE cvr.bActivo = 1
    -- AND cvr.nIdCliente = 369  
    AND dc.nIdStatusCliente = 9 /*VALOR RESIDUAL*/
GROUP BY cvr.nIdValorResidual ;







-- Vin duplicados  -- Pasar a requena si hay valores.
SELECT COUNT(`nIdActivo`) as total, GROUP_CONCAT(nIdActivo), sVin  FROM dat_activo WHERE bActivo = 1 GROUP by `sVin` having total > 1;







  /*OBTIENE LAS OP'S QUE ESTAN PAGADAS Y NO TIENEN FACTURA*/

  (SELECT oop.nIdOpsOP, oop.nIdFactura, oop.nTotal , 'ARRENDAMIENTO', dFecRegistro, sObervaciones FROM  db_besta_management_prod.ops_orden_pago oop
  WHERE (oop.nIdFactura <=0 OR oop.nIdFactura IS NULL)
  AND oop.nIdStatusOp = 2
  AND oop.bActivo = 1
  )
  UNION
  (
  SELECT odgn.nIdOpsGN, odgn.nIdFactura, odgn.nTotal, 'GASTO NOTARIAL', dFecRegistro, sObervaciones FROM  db_besta_management_prod.ops_deposito_gastos_notariales odgn
  WHERE (odgn.nIdFactura <=0 OR odgn.nIdFactura IS NULL)
  AND odgn.nIdStatusOp = 2
  AND odgn.bActivo = 1
  )

  UNION(
  SELECT dpad.nIdPagareAutomaticDetalle, dpad.nMonto ,dpad.nIdFactura, 'PAGARE AUTOMATICO', dFecRegistro, sObervaciones FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad
  WHERE (dpad.nIdFactura <=0 OR dpad.nIdFactura IS NULL )
  AND dpad.nIdStatusOp = 2
  AND dpad.bActivo = 1)
  ORDER BY dFecRegistro DESC
  ;








-- Obtiene las unidades electricas (al dia de hoy 2024-11-20 solo hay 15 registros, de los cuales solo 14 son realmente electricos JAC)
SELECT * FROM `dat_activo` WHERE `bGNV` = 0 AND bActivo = 1;

AVISAR A ALDO HASTA ABAJO

-- Pagare con pago inicial facturados con interes 
#No hay

SELECT dpa.nIdPagareAutomatic, dpad.nIdPagareAutomaticDetalle, df.nIdFactura, dpa.bMontoInicial, dpa.nMontoInicial, dpad.nMonto, 
dpad.jDesglose, dpad.nMonto, df.Total, (df.Total - dpad.nMonto) AS diff, df.nIdStatusFactura 
FROM db_besta_management_prod.dat_pagare_automatic dpa
	JOIN db_besta_management_prod.dat_pagare_automatic_detalle dpad
		ON dpad.nIdPagareAutomatic = dpa.nIdPagareAutomatic
			AND dpad.nMonto = dpa.nMontoInicial
			AND dpad.bActivo = 1
	JOIN db_besta_management_prod.dat_facturas df 
		ON df.nIdFactura = dpad.nIdFactura 
			AND df.bActivo = 1
	WHERE dpa.bMontoInicial = 1
	AND dpa.nMontoInicial > 0
	 	AND dpa.bActivo = 1
	 	AND dpa.nIdTipoPagareAutomatic = 13 #ESPECIAL
		#AND dpa.dFecRegistro >= "2026-08-10"
	HAVING diff > 1 OR diff < -1;
	
-- UNIDADES REPETIDAS EN DAT_CLIENTES

SELECT GROUP_CONCAT(CONCAT(dc.nIdCliente, " (", csc.sNombre , ") ")) AS nIdClientes, 
dc.nIdActivo, csa.sNombre AS sStatusUnidad, 
COUNT(dc.nIdCliente) AS nClientes 
FROM db_besta_management_prod.dat_cliente dc
	JOIN db_besta_management_prod.dat_activo da
		ON da.nIdActivo = dc.nIdActivo
			AND da.bActivo = 1
	LEFT JOIN db_besta_management_prod.cat_status_cliente csc
		ON csc.nIdStatusCliente = dc.nIdStatusCliente
	LEFT JOIN db_besta_management_prod.cat_status_activo csa
		ON csa.nIdStatusActivo = da.nIdStatus
WHERE dc.nIdActivo IS NOT NULL 
	AND dc.bActivo = 1
GROUP BY dc.nIdActivo
HAVING nClientes > 1
;


#Facturas que están en mas de una op

SELECT 
	oop.nIdFactura,
	GROUP_CONCAT(CONCAT(dc.sNombre,' ',dc.sApellidoP,' ',dc.sApellidoM,' ')) AS  sNombreClientes,  
	GROUP_CONCAT(oop.nIdOpsOP) AS nIdOps, 
	COUNT(oop.nIdOpsOP) AS nOps,
	df.Folio,
	df.Serie,
	df.Date
FROM db_besta_management_prod.ops_orden_pago oop
JOIN db_besta_management_prod.dat_cliente dc
	ON dc.nIdCliente = oop.nIdCliente
JOIN db_besta_management_prod.dat_facturas df
	ON df.nIdFactura = oop.nIdFactura
WHERE oop.bActivo = 1
AND oop.nIdFactura IS NOT NULL AND oop.nIdFactura > 0
GROUP BY oop.nIdFactura
HAVING nOps > 1
;


#Ops pagadas con factura ppd pero sin complemento de pago
 
SELECT dc.nIdCliente, CONCAT(dc.sNombre,' ',dc.sApellidoP,' ',dc.sApellidoM) AS  sNombreCliente, 
oop.nIdOpsOP, oop.nTotal, oop.dFecVencimiento, oop.sObervaciones,
df.nIdFactura, df.Serie, df.Folio, df.Date
FROM db_besta_management_prod.ops_orden_pago oop
JOIN db_besta_management_prod.dat_cliente dc
	ON dc.nIdCliente = oop.nIdCliente
JOIN db_besta_management_prod.dat_facturas df
	ON oop.nIdFactura = df.nIdFactura
		AND df.bActivo = 1
		AND df.PaymentMethod = 'PPD - Pago en parcialidades ó diferido'
LEFT JOIN db_besta_management_prod.dat_complemento_pagos dcp
	ON df.nIdFactura = dcp.nIdFactura
		AND dcp.bActivo = 1
WHERE oop.bActivo = 1
AND oop.nIdStatusOp = 2
AND dcp.nIdComplentoPago IS NULL 
AND oop.dFecVencimiento >= "2025-01-01 00:00:00"
ORDER BY df.Date DESC
;


#Pagos que están en diferentes ops de pagaré auto

SELECT drppa.nIdPagoPaycash, GROUP_CONCAT(drppa.nIdPagareAutomaticDetalle) OpsPagare, COUNT(drppa.nIdPagareAutomaticDetalle) AS counter, GROUP_CONCAT(drppa.dFecRegistro) dates
FROM db_besta_management_prod.dat_relacion_pagos_pagare_automatic drppa 
WHERE drppa.bActivo = 1
GROUP BY drppa.nIdPagoPaycash
HAVING counter > 1
;


#Ops de pagaré que tienen diferentes pagos asociados

SELECT drppa.nIdPagareAutomaticDetalle, GROUP_CONCAT(drppa.nIdPagoPaycash) nIdPagos, COUNT(drppa.nIdPagoPaycash) AS counter, GROUP_CONCAT(drppa.dFecRegistro) dates
FROM db_besta_management_prod.dat_relacion_pagos_pagare_automatic drppa 
WHERE drppa.bActivo = 1
GROUP BY drppa.nIdPagareAutomaticDetalle
HAVING counter > 1
;


  #Pagos que están en diferentes ops
  
  SELECT drpo.nIdPagoPaycash,
  GROUP_CONCAT(drpo.nIdOrdenPago) nIdOps,
  COUNT(drpo.nIdOrdenPago) counter, GROUP_CONCAT(drpo.dFecRegistro) dates
  FROM db_besta_management_prod.dat_relacion_pagos_op drpo 
  WHERE drpo.bActivo = 1
  GROUP BY drpo.nIdPagoPaycash
  HAVING counter > 1
  ;


#Ops que tienen diferentes pagos asociados

SELECT drpo.nIdOrdenPago,
GROUP_CONCAT(drpo.nIdPagoPaycash) nIdPagos,
COUNT(drpo.nIdPagoPaycash) counter, GROUP_CONCAT(drpo.dFecRegistro) dates
FROM db_besta_management_prod.dat_relacion_pagos_op drpo 
WHERE drpo.bActivo = 1
GROUP BY drpo.nIdOrdenPago
HAVING counter > 1
;


#Ops de pagaré automático con la misma factura

SELECT dpa.nIdCliente,
dpad.nIdPagareAutomaticDetalle, dpad.nMonto, dpad.dFecVencimiento, dpad.sObervaciones,
df.nIdFactura, df.Serie, df.Folio, df.Date, COUNT(dpad.nIdFactura) AS counter 
FROM db_besta_management_prod.dat_pagare_automatic_detalle dpad
	JOIN db_besta_management_prod.dat_facturas df
		ON df.nIdFactura = dpad.nIdFactura
			AND df.bActivo = 1
	JOIN db_besta_management_prod.dat_pagare_automatic dpa
		ON dpa.nIdPagareAutomatic = dpad.nIdPagareAutomatic
			AND dpa.bActivo = 1
WHERE 
dpad.nIdFactura IS NOT NULL 
AND dpad.bActivo = 1
GROUP BY dpad.nIdFactura
HAVING counter > 1
;


-- Se corrre los lunes y se le  pasa a Ceci  solo los primero 6 campos. Siempre le digo  : Hola buenos días    le comparto esta info de los clientes que se pueden caer  
   /*******************************************************************************


    * Nombre..........:sp_select_tarea_proceso_caido_pagare_automatic


    * Propósito.......:Obtiene los clientes que cumplan los siguiente ademas de unos calculos extras
 
    - En cuanto se carga la evidencia de contacto se cae
 
    - Si debe una semana y debe pagaré se cae
 
    - Si no debe semanas y debe dos pagares automaticos se cae
 
    - Si en dos dias no se carga documento se avisa que se puede caer
 
 
    * Autor...........:MAMP


    * Empresa.........:Besta


    * Sistema.........:Besta management


    * Fecha creación.....:2024-01-02


    * Ejemplo de ejecución:


    *    CALL sp_select_tarea_proceso_caido_pagare_automatic();


    *


    ********************************************************************************/
 
-- Estaditas extras
 
    SELECT T1.*,
 
           (SELECT CONCAT_WS(' ',dc.sNombre,dc.sApellidoP,dc.sApellidoM)  FROM db_besta_management_prod.dat_cliente AS dc WHERE nIdCliente =  T1.nIdCliente) AS sNombreCompleto,


           IFNULL((SELECT nIdFactura FROM db_besta_management_prod.ops_orden_pago WHERE nIdCliente  =  T1.nIdCliente  AND nIdFactura > 0  AND nIdStatusOp NOT IN (3,2) ORDER BY nIdOpsOP DESC LIMIT 1),0) AS nIdFacturaCancelar,


           (SELECT COUNT(nTotal)


            FROM db_besta_management_prod.ops_orden_pago AS oop


            WHERE oop.nIdCliente = T1.nIdCliente


              AND oop.nIdStatusOp = 4 -- VENCIDA


              AND oop.bActivo = 1)                                AS numOrdenesVencidos,
 
           (SELECT IFNULL(SUM(nTotal), 0)


            FROM db_besta_management_prod.ops_orden_pago AS oop


            WHERE oop.nIdCliente = T1.nIdCliente


              AND oop.nIdStatusOp = 4 -- VENCIDA


              AND oop.bActivo = 1)                                AS montoOrdenesPagoVencido,
 
           (SELECT IFNULL(SUM(nTotal), 0)


            FROM db_besta_management_prod.ops_orden_pago AS oop


            WHERE oop.nIdCliente = T1.nIdCliente


              AND oop.nIdStatusOp = 1 -- PENDIENTE


              AND oop.bActivo = 1)                                AS montoOrdenesPagoPendiente,
 
           (SELECT COUNT(dpad.nMonto)


            FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad


            WHERE dpad.bActivo = 1


              AND dpad.nIdStatusOp = 4 -- VENCIDO


              AND nIdPagareAutomatic = T1.nIdPagareAutomatic)     AS numPagosPagareAutomaticoVencidos,
 
           (SELECT IFNULL(SUM(dpad.nMonto), 0)


            FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad


            WHERE dpad.bActivo = 1


              AND dpad.nIdStatusOp = 4 -- VENCIDO


              AND nIdPagareAutomatic = T1.nIdPagareAutomatic)     AS montoPagareAutomaticoCalculadoVencido,
 
 
           (SELECT IFNULL(SUM(dpad.nMonto), 0)


            FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad


            WHERE dpad.bActivo = 1


              AND dpad.nIdStatusOp = 1 -- PENDIENTE


              AND nIdPagareAutomatic = T1.nIdPagareAutomatic)     AS montoPagareAutomaticoCalculadoPendiente,
 
           (SELECT IFNULL(SUM(dpad.nMonto), 0)


            FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad


            WHERE dpad.bActivo = 1


              AND dpad.nIdStatusOp = 2 -- PAGADA


              AND nIdPagareAutomatic = T1.nIdPagareAutomatic)     AS montoPagareAutomaticoCalculadoPagado,
 
           (SELECT nMontoPagoFinal + nMontoInicial


            FROM db_besta_management_prod.dat_pagare_automatic AS dpa


            WHERE bActivo = 1


              AND dpa.nIdPagareAutomatic = T1.nIdPagareAutomatic) AS montoPagareAutomaticoReal
 
    FROM (


--  En cuanto se carga la evidencia de contacto se cae y confirma administracion agregara el archivo exista la liga


-- nIdCliente -> 1138


             SELECT nIdCliente,


                    nIdPagareAutomatic,


                    '0'                                                                                                            AS puedeCaerCliente,


                    1                                                                                                              AS caso,


                    'Cuanto se carga la evidencia de contacto se cae' AS sDescripcionMotivoCaido


             FROM db_besta_management_prod.dat_pagare_automatic AS dpa


             WHERE nIdStatusPagareAutomatic = 16 -- EL CLIENTE NO SE PRESENTO FIRMAR


               AND sFileEvidenciaContacto IS NULL


               AND dpa.bActivo = 1
 
             UNION
 
             --   Si debe una semana y debe pagaré se cae


             -- nIdCliente -> 200
 
             SELECT nIdCliente,


                    nIdPagareAutomatic,


                    '0'                                       AS puedeCaerCliente,


                    2                                         AS caso,


                    'Si debe una semana y debe pagaré se cae' AS sDescripcionMotivoCaido


             FROM (SELECT dpa.nIdCliente,


                          dpa.nIdPagareAutomatic,


                          (SELECT COUNT(*)


                           FROM ops_orden_pago AS oop


                           WHERE oop.nIdCliente = dpa.nIdCliente


                             AND oop.bActivo = 1


                             AND oop.nIdStatusOp = 4)   AS countDebeSemanas,


                          (SELECT COUNT(*)


                           FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad2


                           WHERE dpad2.nIdPagareAutomatic = dpa.nIdPagareAutomatic


                             AND dpad2.bActivo = 1


                             AND dpad2.nIdStatusOp = 4) AS countDebePagares


                   FROM db_besta_management_prod.dat_pagare_automatic AS dpa


                   WHERE nIdStatusPagareAutomatic = 14 -- EN PROCESO


                  ) AS T1


             WHERE countDebeSemanas = 1


               AND countDebePagares = 1
 
             UNION
 
             -- Si no debe semanas y debe dos pagares automaticos se cae


             -- 1014
 
             SELECT nIdCliente,


                    nIdPagareAutomatic,


                    '0'                                                        AS puedeCaerCliente,


                    3                                                          AS caso,


                    'Si no debe semanas y debe dos pagares automaticos se cae' AS sDescripcionMotivoCaido


             FROM (SELECT dpa.nIdCliente,


                          dpa.nIdPagareAutomatic,


                          (SELECT COUNT(*)


                           FROM db_besta_management_prod.ops_orden_pago AS oop


                           WHERE oop.nIdCliente = dpa.nIdCliente


                             AND oop.nIdStatusOp = 4)   AS countDebeSemanas,


                          (SELECT COUNT(*)


                           FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad2


                           WHERE dpad2.nIdPagareAutomatic = dpa.nIdPagareAutomatic


                             AND dpad2.bActivo = 1


                             AND dpad2.nIdStatusOp = 4) AS countDebePagares


                   FROM db_besta_management_prod.dat_pagare_automatic AS dpa


                   WHERE nIdStatusPagareAutomatic = 14 -- EN PROCESO


                  ) AS T1


             WHERE countDebeSemanas = 0


               AND countDebePagares = 2
 
             UNION
 
             -- Si en dos dias no se carga documento se avisa que se puede caer


             -- nIdCliente -> 374
 
             SELECT nIdCliente,


                    nIdPagareAutomatic,


                    (DATE_ADD(dFecMod, INTERVAL 2 DAY) <= NOW())                      AS puedeCaerCliente,


                    4                                                                 AS caso,


                    'Si en dos dias no se carga documento se avisa que se puede caer' AS sDescripcionMotivoCaido


             FROM db_besta_management_prod.dat_pagare_automatic


             WHERE bActivo = 1


               AND sFileEvidenciaContacto IS NULL


               AND sFilePagareFirmado IS NULL


               AND nIdStatusPagareAutomatic = 5) AS T1 LEFT JOIN db_besta_management_prod.dat_cliente AS dc ON T1.nIdCliente = dc.nIdCliente WHERE dc.nIdStatusCliente != 7;



-- UNIDADES CARGADAS EN ENTERPRISE QUE AUN NO LLEGAN A BM

SELECT csaif.sNombre AS sStatusFactura,daif.* FROM db_besta_management_prod.dat_activo_info_factura daif 
JOIN db_besta_management_prod.cat_status_activo_info_factura csaif
ON csaif.nIdStatusActInfFac = daif.nIdStatusActInfFac
LEFT JOIN db_besta_management_prod.dat_activo da
ON da.nIdActivo = daif.nIdActivo
WHERE daif.bActivo = 1
AND (da.nIdActivo IS NULL  OR daif.nIdActivo IS NULL)
ORDER BY `daif`.`nIdActInfFac` ASC;



-- Ops Pagaré automático pagadas sin factura
 
SELECT dpa.nIdPagareAutomatic,ctpa.sNombre ,dpad.*
 
FROM db_besta_management_prod.dat_pagare_automatic dpa
 
JOIN db_besta_management_prod.cat_tipo_pagare_automatic ctpa
ON ctpa.nIdTipoPagareAutomatic = dpa.nIdTipoPagareAutomatic
AND ctpa.bFactura = 1
 
JOIN db_besta_management_prod.dat_pagare_automatic_detalle dpad
ON dpad.nIdPagareAutomatic = dpa.nIdPagareAutomatic
AND dpad.bActivo = 1 
AND dpad.nIdFactura IS NULL
AND dpad.nIdStatusOp = 2 
AND dpa.nIdTipoPagareAutomatic != 10
 
WHERE dpa.bActivo = 1 ;
