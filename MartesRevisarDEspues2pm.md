#Revisar ops 

SELECT dc.nIdCliente, 
	CONCAT(dc.sNombre,' ',dc.sApellidoP,' ',dc.sApellidoM) AS  sNombreCliente,
	dc.nIdStatusCliente,
	oop.nIdOpsOP,
	oop.nIdStatusOp
FROM db_besta_management_prod.dat_cliente dc 
	LEFT JOIN db_besta_management_prod.ops_orden_pago oop
		ON oop.nIdCliente = dc.nIdCliente
		 	AND oop.dFecVencimiento = "2025-10-20 00:00:00"
			AND oop.bActivo = 1  
WHERE dc.nIdStatusCliente IN(2,3,4,9)
	AND dc.bActivo = 1
	AND dc.dFechaInicio <= DATE_FORMAT(NOW(), '%Y-%m-%d')
	AND dc.dFechaVencimiento >= DATE_FORMAT(NOW(), '%Y-%m-%d')
	;


#Revisar ops plan de pagos

SELECT dc.nIdCliente, 
	CONCAT(dc.sNombre,' ',dc.sApellidoP,' ',dc.sApellidoM) AS  sNombreCliente,
	dc.nIdStatusCliente,
	oop.nIdOpsOP,
	(SELECT dpp.dFecVencimiento FROM db_besta_management_prod.dat_plan_pago dpp
		WHERE dpp.nIdCliente = dc.nIdCliente
			AND dpp.bActivo = 1
			AND dpp.nIdStatusPlanPago = 1
	ORDER BY dpp.dFecVencimiento DESC LIMIT 1) AS PlanPagoDFecVencimiento
	
FROM db_besta_management_prod.dat_cliente dc 
	LEFT JOIN db_besta_management_prod.ops_orden_pago oop
		ON oop.nIdCliente = dc.nIdCliente
		 	AND oop.dFecVencimiento = "2025-10-20 00:00:00"
			AND oop.bActivo = 1  
WHERE dc.nIdStatusCliente IN(/*2,3,*/4/*,9*/)
	AND dc.bActivo = 1
	AND dc.dFechaInicio <= DATE_FORMAT(NOW(), '%Y-%m-%d')
	AND dc.dFechaVencimiento >= DATE_FORMAT(NOW(), '%Y-%m-%d')
	;
