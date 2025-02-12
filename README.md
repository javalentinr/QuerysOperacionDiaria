# Querys de operaciones diarias.


 ### Se tienen que pasar a Aldo.

Query de los martes despues de comer.
``` SqlQuery
 -- Consulta que obtiene la cantidad de OP's generadas y la cantidad que deberian estar generadas
 
 (
  SELECT 
		dc.nIdCliente,
		dc.nIdActivo,
		dc.nIdStatusCliente,
        UPPER (csc.sNombre ) AS sStatusCliente,
		cop.nIdOrdenPago, 
		date_format(cop.dFecVencimiento, "%Y-%m-%d") AS dFecVencimiento,
		cop.nTotal   AS nTotal,
		CONCAT(dc.sNombre,                
               IF(dc.sApellidoP IS NULL OR dc.sApellidoP = '','',CONCAT(' ',dc.sApellidoP)),                 
               IF(dc.sApellidoM IS NULL OR dc.sApellidoM = '','',CONCAT(' ',dc.sApellidoM))               
              ) AS sNombreCliente,		 
		/*dc.nIdRegimen ,
		dc.nIdUsoCfdi ,
		IFNULL( cuc.sNombre ,0) AS sUsoCfdi,
		IFNULL(crf.sNombre,0) AS sRegimenFiscal,
        IF( CONCAT(YEAR(NOW()),WEEKOFYEAR(NOW())) =  CONCAT(YEAR(cop.dFecVencimiento),WEEKOFYEAR(cop.dFecVencimiento)) ,1,0 ) AS NOTIFICAR,   
        ELT(WEEKDAY(cop.dFecVencimiento) + 1, 'Lunes', 'Martes', 'Miercoles', 'Juevez', 'Viernes', 'Sabado', 'Domingo') AS DIA_SEMANA,*/         
         cp.dFecInicio,
         cp.dFecVencimiento,
         TIMESTAMPDIFF(WEEK,cp.dFecInicio,cp.dFecVencimiento) AS plazoArrendamiento,
         TIMESTAMPDIFF(WEEK,cp.dFecInicio,now()) AS plazoActualArrendamientoCalculo,
         (
             SELECT COUNT(oop.dFecVencimiento )
             FROM ops_orden_pago oop 
             WHERE oop.nIdCliente = dc.nIdCliente
             AND oop.nIdStatusOp != 3
             AND (oop.dFecVencimiento >= cp.dFecInicio AND oop.dFecVencimiento <= cp.dFecVencimiento)
             AND oop.bActivo = 1
         ) AS totalOPSActual,             
             
         DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") hace8semanas,
     
      IF( DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") < cp.dFecInicio , cp.dFecInicio ,  DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") )  hace8semanas,
             
         TIMESTAMPDIFF(WEEK,      
                       
                        IF( DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") < cp.dFecInicio , cp.dFecInicio ,  DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") ) 
                       
                       ,now()) AS semanas8semanasCalculo,
         (
                  SELECT COUNT(oop.dFecVencimiento )
                  FROM ops_orden_pago oop 
                  WHERE oop.nIdCliente = dc.nIdCliente
                  AND oop.nIdStatusOp != 3
                  AND (oop.dFecVencimiento >= DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") 
                       AND oop.dFecVencimiento <=now())
                  AND oop.bActivo = 1
              ) AS totalOPS8semanas
	
	FROM db_besta_management_prod.dat_cliente dc
    	JOIN db_besta_management_prod.cat_status_cliente AS csc
			ON csc.nIdStatusCliente  = dc.nIdStatusCliente
	
		JOIN db_besta_management_prod.cfg_producto cp 
		ON cp.nIdCliente = dc.nIdCliente
      AND cp.bActivo = 1
		
		JOIN db_besta_management_prod.cfg_orden_pago cop 
		ON cop.nIdCfgProducto = cp.nIdCfgProducto 
      AND cop.bActivo = 1
		
		LEFT JOIN db_besta_management_prod.cat_uso_cfdi cuc
		ON cuc.nIdUsoCfdi = dc.nIdUsoCfdi
		AND cuc.bActivo = 1

		LEFT JOIN db_besta_management_prod.cat_regimen_fiscal crf
		ON crf.nIdRegimen = dc.nIdRegimen
		AND crf.bActivo = 1
		
		LEFT JOIN db_besta_management_prod.cat_colonia ccol
		ON ccol.nIdColonia = dc.nIdColonia
	                  
	WHERE   dc.nIdStatusCliente IN (2,3,4,5) AND dc.bActivo = 1
    HAVING semanas8semanasCalculo != totalOPS8semanas
    ) UNION
    
    (
    
        SELECT 

		dc.nIdCliente,
		dc.nIdActivo,
		dc.nIdStatusCliente,
        UPPER (csc.sNombre ) AS sStatusCliente,
		cvr.nIdValorResidual, 
		date_format(cvr.dFecVencimiento, "%Y-%m-%d") AS dFecVencimiento,
		cvr.nTotal   AS nTotal,
		CONCAT(dc.sNombre, ' ', dc.sApellidoP, ' ', dc.sApellidoM) AS sNombreCliente,
		/*dc.sRFC,
		ccol.sCodigoPostal AS nCodigoPostalCliente,
		cvr.nInteresValorResidual,
		cvr.nSeguro,
        cvr.nGPS,
        cvr.nValorOP,
        cvr.nTotal,
		dc.nIdRegimen ,
		dc.nIdUsoCfdi ,
		IFNULL( cuc.sNombre ,0) AS sUsoCfdi,
		IFNULL(crf.sNombre,0) AS sRegimenFiscal,
        dc.nIdCartera,
        IF( CONCAT(YEAR(NOW()),WEEKOFYEAR(NOW())) =  CONCAT(YEAR(cvr.dFecVencimiento),WEEKOFYEAR(cvr.dFecVencimiento)) ,1,0 ) AS NOTIFICAR,*/
        
         cvr.dFechaInicio,
         cvr.dFecVencimiento,
         TIMESTAMPDIFF(WEEK,cvr.dFechaInicio,cvr.dFecVencimiento) AS plazoArrendamiento,
         TIMESTAMPDIFF(WEEK,cvr.dFechaInicio,now()) AS plazoActualArrendamientoCalculo,
         (
             SELECT COUNT(oop.dFecVencimiento )
             FROM ops_orden_pago oop 
             WHERE oop.nIdCliente = dc.nIdCliente
             AND oop.nIdStatusOp != 3
             AND (oop.dFecVencimiento >= cvr.dFechaInicio AND oop.dFecVencimiento <= cvr.dFecVencimiento)
             AND oop.bActivo = 1
         ) AS totalOPSActual,             
             
         DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") hace8semanas,
         IF( DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") < cvr.dFechaInicio , cvr.dFechaInicio ,  DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") )  hace8semanas,
             
         TIMESTAMPDIFF(WEEK,
                       
                        IF( DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") < cvr.dFechaInicio , cvr.dFechaInicio ,  DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") )
                       
                       
                       ,now()) AS semanas8semanasCalculo,
         (
                  SELECT COUNT(oop.dFecVencimiento )
                  FROM ops_orden_pago oop 
                  WHERE oop.nIdCliente = cvr.nIdCliente
                  AND oop.nIdStatusOp != 3
                  AND (oop.dFecVencimiento >= DATE_FORMAT(DATE_ADD(now(), INTERVAL -8 WEEK), "%Y-%m-%d 00:00:00") 
                       AND oop.dFecVencimiento <=now())
                  AND oop.bActivo = 1
              ) AS totalOPS8semanas
	
	FROM db_besta_management_prod.dat_cliente dc		
    	JOIN db_besta_management_prod.cat_status_cliente AS csc
			ON csc.nIdStatusCliente  = dc.nIdStatusCliente
	
		JOIN db_besta_management_prod.cfg_valor_residual cvr
		ON cvr.nIdCliente = dc.nIdCliente
		AND cvr.bActivo = 1
		
		LEFT JOIN db_besta_management_prod.cat_uso_cfdi cuc
		ON cuc.nIdUsoCfdi = dc.nIdUsoCfdi
		AND cuc.bActivo = 1

		LEFT JOIN db_besta_management_prod.cat_regimen_fiscal crf
		ON crf.nIdRegimen = dc.nIdRegimen
		AND crf.bActivo = 1
		
		LEFT JOIN db_besta_management_prod.cat_colonia ccol
		ON ccol.nIdColonia = dc.nIdColonia
	                  
	WHERE 
      dc.nIdStatusCliente IN (9) AND dc.bActivo = 1
    HAVING semanas8semanasCalculo != totalOPS8semanas
    
    
    ) ORDER BY sNombreCliente;

```


Querys Diarios que se les paso a aldo. 

``` SqlQuery
-- CHECAR QUE EL CLIENTE USUARIO TENGA EMAIL ASIGNADO PARA EL ENVIO DE PAGARÉ AUTOMÁTICO
SELECT dpa.nIdPagareAutomatic,
       CONCAT_WS(' ', du.sNombre, du.sApellidoP, du.sApellidoM) AS sNombreCliente,
       dpa.nIdCliente,
       sCorreo                                                  AS sCorreoDU,
       dc.eCorreo,
       sCorreo = dc.eCorreo                                     AS correIgual,
       dpa.nIdStatusPagareAutomatic,
       cspa.sNombre,
       dc.nIdStatusCliente,
       ccc.sNombre,
       dpa.nIdTipoPagareAutomatic,
       ctpa.sNombre,
       dpa.dFecRegistro,
       dpa.dFecMod
FROM db_besta_management_prod.dat_pagare_automatic AS dpa
         LEFT JOIN db_besta_management_prod.dat_usuarios AS du
                   ON dpa.nIdCliente = du.nIdCliente
                       AND du.bActivo = 1
         LEFT JOIN db_besta_management_prod.cat_status_pagare_automatic AS cspa
                   ON dpa.nIdStatusPagareAutomatic = cspa.nIdStatusPagareAutomatic
                       AND cspa.bActivo = 1
         LEFT JOIN db_besta_management_prod.dat_cliente AS dc
                   ON dpa.nIdCliente = dc.nIdCliente
                       AND dc.bActivo = 1
         LEFT JOIN db_besta_management_prod.cat_status_cliente AS ccc
                   ON dc.nIdStatusCliente = ccc.nIdStatusCliente
                       AND ccc.bActivo = 1
         LEFT JOIN db_besta_management_prod.cat_tipo_pagare_automatic AS ctpa
                   ON dpa.nIdTipoPagareAutomatic = ctpa.nIdTipoPagareAutomatic
                       AND ctpa.bActivo = 1
WHERE 1
  AND dpa.nIdStatusPagareAutomatic NOT IN (2, 15, 14, 17)
  AND dc.bActivo = 1
  AND (du.sCorreo NOT LIKE '%_PROSPECTO' AND du.sCorreo NOT LIKE '%_PROSPE')
GROUP BY dpa.nIdPagareAutomatic
ORDER BY nIdPagareAutomatic DESC;


-- checar que ops de pagare automatico no an sido facturadas no olvidar que existen tipos de pagares que no deben de facturar
SELECT dpad.nIdPagareAutomaticDetalle,
       dpad.nIdPagareAutomatic,
       dpad.nIdStatusOp,
       dpad.nIdFactura,
       dpp.*,
       dpa.nIdCliente,
       dc.nIdCliente,
       dc.sCIELegacy,
       CONCAT(dc.sNombre,
              IF(dc.sApellidoP IS NULL OR dc.sApellidoP = '', '', CONCAT(' ', dc.sApellidoP)),
              IF(dc.sApellidoM IS NULL OR dc.sApellidoM = '', '', CONCAT(' ', dc.sApellidoM))
       )                                                        AS sNombre,
       dc.sRFC,
       CONCAT_WS(' ', du.sNombre, du.sApellidoP, du.sApellidoM) AS nombreVendedor,
       dc.bFacturaPublicoGeneral,
       dc.nIdStatusCliente,
       ctpa.bFactura,
       dc.nIdUsoCfdi,
       dc.nIdCartera,
       dc.nIdRegimen,
       dc.nIdColonia,
       dc.sConstanciaFiscal                                     AS CSFdc,
       dd.sFile                                                 AS CSFDocs
FROM db_besta_management_prod.dat_relacion_pagos_pagare_automatic AS drppa
         LEFT JOIN db_besta_management_prod.dat_pagare_automatic_detalle AS dpad
                   ON drppa.nIdPagareAutomaticDetalle = dpad.nIdPagareAutomaticDetalle
         LEFT JOIN db_besta_management_prod.dat_pagos_paycash AS dpp
                   ON drppa.nIdPagoPaycash = dpp.nIdPagoPaycash
         LEFT JOIN db_besta_management_prod.dat_pagare_automatic AS dpa
                   ON dpad.nIdPagareAutomatic = dpa.nIdPagareAutomatic
         LEFT JOIN db_besta_management_prod.dat_cliente AS dc
                   ON dpa.nIdCliente = dc.nIdCliente
         LEFT JOIN db_besta_management_prod.dat_usuarios AS du
                   ON dc.nIdVendedor = du.nIdUsuario
         LEFT JOIN db_besta_management_prod.cat_tipo_pagare_automatic ctpa
                   ON dpa.nIdTipoPagareAutomatic = ctpa.nIdTipoPagareAutomatic
         LEFT JOIN db_besta_management_prod.dat_documentos dd
                   ON dd.nIdCliente = dc.nIdCliente
                       AND dd.nIdTipoDocumento = 1 -- Constancia fiscal
WHERE 1
  AND dpad.nIdStatusOp IN (2)
  AND dc.nIdStatusCliente <> 7 -- CAIDO
  AND dpad.nIdFactura IS NULL;


  -- CONSULTA PARA SABER LOS PAGOS QUE SE DEBEN DE HACER ANTES DE HOY DE PAGARE AUTOMATICO
SELECT dpad.nIdPagareAutomaticDetalle,
       dpa.nIdPagareAutomatic,
       dpad.sReferenciaPaycash,
       dpad.dFecVencimiento,
       dpad.nIdStatusOp,
       dpad.nMonto,
       dpa.nIdCliente,
       CONCAT_WS(' ', dc.sNombre, dc.sApellidoP, dc.sApellidoM) AS sNombreCliente,
       dpa.nIdUserAsignado,
       cspa.sNombre AS NombreStatus,
       ctpa.sNombre AS NombreTipoPagare
FROM db_besta_management_prod.dat_pagare_automatic_detalle AS dpad
LEFT JOIN db_besta_management_prod.dat_pagare_automatic AS dpa
       ON dpad.nIdPagareAutomatic = dpa.nIdPagareAutomatic
      AND dpa.nIdStatusPagareAutomatic NOT IN (2, 15, 17) -- 2-CANCELADO, 15-LIQUIDADO, 17-DEUDA EXTRAORDINARIA
      AND dpa.bActivo = 1
LEFT JOIN db_besta_management_prod.dat_cliente AS dc
       ON dpa.nIdCliente = dc.nIdCliente
      AND dc.bActivo = 1
LEFT JOIN db_besta_management_prod.cat_status_pagare_automatic AS cspa
       ON dpa.nIdStatusPagareAutomatic = cspa.nIdStatusPagareAutomatic
      AND cspa.bActivo = 1
LEFT JOIN db_besta_management_prod.cat_tipo_pagare_automatic AS ctpa
       ON dpa.nIdTipoPagareAutomatic = ctpa.nIdTipoPagareAutomatic
      AND ctpa.bActivo = 1
WHERE dpad.bActivo = 1
  AND dc.nIdStatusCliente NOT IN (7) -- 7-CAIDO
  AND dpad.dFecVencimiento <= CURDATE()
  AND dpad.nIdStatusOp IN (1, 4) -- 1-PENDIENTE, 4-VENCIDA
ORDER BY dpad.nIdPagareAutomatic;

-- consulta de clientes caidos pagare automatico
SELECT T1.*,
       ctpa.sNombre AS tipoPagare,
       (SELECT CONCAT_WS(' ', dc.sNombre, dc.sApellidoP, dc.sApellidoM)
       FROM db_besta_management_prod.dat_cliente AS dc
       WHERE nIdCliente = T1.nIdCliente)                     AS sNombreCompleto,
       IFNULL((SELECT nIdFactura
       FROM db_besta_management_prod.ops_orden_pago
       WHERE nIdCliente = T1.nIdCliente
         AND nIdFactura > 0
         AND nIdStatusOp NOT IN (3, 2)
       ORDER BY nIdOpsOP DESC
       LIMIT 1), 0)                                          AS nIdFacturaCancelar,
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
                '0'                                               AS puedeCaerCliente,
                1                                                 AS caso,
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
                      FROM db_besta_management_prod.ops_orden_pago AS oop
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
           AND nIdStatusPagareAutomatic = 5) AS T1

         LEFT JOIN db_besta_management_prod.dat_cliente AS dc ON T1.nIdCliente = dc.nIdCliente
         LEFT JOIN db_besta_management_prod.dat_pagare_automatic AS dpa3 ON T1.nIdPagareAutomatic = dpa3.nIdPagareAutomatic
    AND dpa3.bActivo = 1

         LEFT JOIN db_besta_management_prod.cat_tipo_pagare_automatic AS ctpa ON dpa3.nIdTipoPagareAutomatic = ctpa.nIdTipoPagareAutomatic
    WHERE dc.nIdStatusCliente != 7;

-- Estadisticas por estatus de pagare automatico
SELECT nIdStatusPagareAutomatic,
       cspa.sNombre,
       cspa.sDescripcion,
       (SELECT COUNT(*)
       FROM db_besta_management_prod.dat_pagare_automatic AS dpa
                LEFT JOIN db_besta_management_prod.dat_cliente AS dc
                          ON dpa.nIdCliente = dc.nIdCliente
       WHERE dpa.bActivo = 1
         AND dc.bActivo = 1
         AND dpa.nIdStatusPagareAutomatic = cspa.nIdStatusPagareAutomatic
         AND nIdStatusCliente NOT IN (7, 6, 5)) AS numPagaresClientesActivos,
       (SELECT COUNT(*)
       FROM db_besta_management_prod.dat_pagare_automatic AS dpa
                LEFT JOIN db_besta_management_prod.dat_cliente AS dc
                          ON dpa.nIdCliente = dc.nIdCliente
       WHERE dpa.bActivo = 1
         AND dc.bActivo = 1
         AND dpa.nIdStatusPagareAutomatic = cspa.nIdStatusPagareAutomatic
         AND nIdStatusCliente = 7)              AS numPagaresClientesCaidos
FROM db_besta_management_prod.cat_status_pagare_automatic AS cspa
WHERE bActivo = 1;

-- Numero de pagare automatico segun el tipo 
SELECT ctpa.nIdTipoPagareAutomatic,
       ctpa.sNombre,
       (SELECT COUNT(dpa.nIdPagareAutomatic)
       FROM db_besta_management_prod.dat_pagare_automatic AS dpa
       WHERE dpa.nIdTipoPagareAutomatic = ctpa.nIdTipoPagareAutomatic) AS nNumPagares
FROM db_besta_management_prod.cat_tipo_pagare_automatic AS ctpa;
```

EDGAR

``` SqlQuery

-- Query de ordenes que quieren estar pagadas pero no sale factura
SELECT * FROM db_besta_management_prod.ops_orden_pago WHERE `bEnviadoFact` = 1 AND `nIdStatusOP` ORDER BY `nIdStatusOP` DESC;


-- Facturas sin relaciÓn de OP arr. plan de pago, VR,  de Gastos notariales, pagare automatico
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
AND YEAR(df.Date) = 2025

AND df.nIdCliente NOT IN (1381) -- facturas son las OP que pidieron ayer que se inactivara del JESUS MARIA LOPEZ MAZZO, el cliente de Salvador que cayeron y le dieron otro carro, por eso no aparecen relacionadas
;

-- Consulta si existen complementos duplicados.
SELECT
    COUNT(*) AS nTotalComplemento,
    nIdFactura,
    `dFecRegistro`
FROM
    db_besta_management_prod.dat_complemento_pagos
WHERE
    bActivo = 1
GROUP BY
    `nIdFactura`
HAVING
    nTotalComplemento > 1 AND YEAR(dFecRegistro) = 2025
ORDER BY
    `dat_complemento_pagos`.`dFecRegistro` ASC;


    -- Obtiene las OP sin pagar con factura pagada

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
	);

-- Facturas de gastos notariales sin generar
SELECT
    odgn.nIdOpsGN,
    odgn.nIdCliente,
    CONCAT(dc.sNombre,' ',dc.sApellidoP,' ',dc.sApellidoM) AS  sNombreCliente	,
    odgn.sReferenciaPaycash,
    SUBSTRING(odgn.dFecRegistro,1,10) AS dFecha,
    SUBSTRING(odgn.dFecRegistro,12,9) AS dHora
FROM
    db_besta_management_prod.`ops_deposito_gastos_notariales` odgn
    JOIN db_besta_management_prod.dat_cliente dc
	ON dc.nIdCliente = odgn.nIdCliente 
	AND dc.bActivo = 1  
WHERE
    odgn.`nIdFactura` IS NULL AND odgn.`nIdStatusOp` = 2
ORDER BY
    odgn.`nIdFactura` ASC;

-- Gastos notariales sin relacion de pago (Revision de factura)
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
    AND drpogn.nIdOpPagoGN IS NULL;

-- Consulta para obtener usuarios sin idcliente

SELECT * FROM db_besta_management_prod.dat_usuarios WHERE `nIdTipoUsuario` = 2 AND `nIdCliente` IS NULL;

-- Consulta que obtiene las OPs de VR que no se han conciliado
SELECT dpp.*
FROM db_besta_management_prod.dat_pagos_paycash dpp
    LEFT JOIN db_besta_management_prod.dat_relacion_pagos_op dro
    ON dro.nIdPagoPaycash = dpp.nIdPagoPaycash
    AND dro.bActivo = 1
	
	
WHERE dpp.bActivo = 1
AND dro.nIdOpPago IS NULL
AND dpp.sReference IN (SELECT sReferenciaPaycash FROM db_besta_management_prod.`cfg_valor_residual` WHERE  bActivo = 1 AND dFecVencimiento >= '2026-01-01' )
;


-- Consulta para buscar un texto dentro de PS´S y Funciones: 
SELECT  ROUTINE_NAME
FROM information_schema.routines 
WHERE routine_definition LIKE '%hst_dat_activo_cliente%' 
ORDER BY routine_name;

-- Consulta para obtener pagos realizados en el mismo día
SELECT *,COUNT(dDate) AS TOTAL FROM db_besta_management_prod.dat_pagos_paycash WHERE dDate >= '2025-06-25' GROUP BY sReference,dDate HAVING TOTAL > 1; 
``` 


### HASH DE CONTRASENA

Para password de enterprise.

``` SqlQuery
    SELECT SHA1(CONCAT("B3st42023#", "Ceniceros1967"))
``` 
