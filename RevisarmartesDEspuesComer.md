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
    AND totalOPSActual < semanas8semanasCalculo OR semanas8semanasCalculo < totalOPS8semanas
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
    AND totalOPSActual < semanas8semanasCalculo OR semanas8semanasCalculo < totalOPS8semanas
    
    ) ORDER BY sNombreCliente 
	 
	 ;
