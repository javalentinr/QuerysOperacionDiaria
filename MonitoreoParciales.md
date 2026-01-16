SELECT 
     IF( CONCAT(YEAR(NOW()),WEEKOFYEAR(NOW())) = 
     CONCAT(YEAR(cop.dFecVencimiento),WEEKOFYEAR(cop.dFecVencimiento)) ,1,0 ) AS VALIDATION1,
     CONCAT(dc.sNombre,                
               IF(dc.sApellidoP IS NULL OR dc.sApellidoP = '','',CONCAT(' ',dc.sApellidoP)),                 
               IF(dc.sApellidoM IS NULL OR dc.sApellidoM = '','',CONCAT(' ',dc.sApellidoM))               
              ) AS sNombreCliente
     #INTO @isTheLastWeek
FROM db_besta_management_prod.dat_cliente dc
	JOIN db_besta_management_prod.cfg_producto cp
	ON cp.nIdCliente = dc.nIdCliente
   AND cp.bActivo = 1
	JOIN db_besta_management_prod.cfg_orden_pago cop 
	ON cop.nIdCfgProducto = cp.nIdCfgProducto 
   AND cop.bActivo = 1
WHERE dc.nIdStatusCliente IN (2,3,4,5)
HAVING VALIDATION1 = 1

;
