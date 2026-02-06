Validacion INE aval manual

CALL  sp_update_ine_aval_prospecto(603,'Edgar','Ramos','Lopez','Siempre viva 123 CP90300','RALE970831HCLMND09', '1997-08-31','RMLNED97083105H500', '143331142', '25', 1,1,1);

	IN `PO_nIdUsuario` INT,
	IN `PO_sNombre` VARCHAR(50),
	IN `PO_sApellidoP` VARCHAR(30),
	IN `PO_sApellidoM` VARCHAR(30),
	IN `PO_sCalle` VARCHAR(255),
	IN `PO_sCURP` VARCHAR(18),
	IN `PO_dFechaNacimiento` DATE,
	IN `PO_sClaveElector` VARCHAR(30),
	IN `PO_sFolioINE` VARCHAR(30),
	IN `PO_sEdad` VARCHAR(255),
	IN `PO_nIdGenero` INT,
	IN `PO_nIdPaisNacimiento` INT,
	IN `PO_nIdStatusDocumento` INT



Validacion CSF manual

CALL sp_update_constancia_prospecto(1984, 2, '44969',"CUAJ8605123K5","Régimen de las Actividades Empresariales con ingresos a través de Plataformas Tecnológicas.", "JESUS GERARDO DE LA CRUZ ALVAREZ");

	IN `PO_nIdUsuario` INT,
	IN `PO_nIdStatusDocumento` INT,
	IN `PO_sCodigoPostal` VARCHAR(6),
	IN `PO_sRFC` VARCHAR(13),
	IN `PO_sRegimenFiscal` VARCHAR(100),
	IN `PO_sRazonSocial` VARCHAR(50)

Validacion manual comprobantes Prosp-Aval

CALL  sp_update_comprobante_prospecto(603,2, 'AVAL- PROSP','Nombre cliente','Calle cliente');
	IN `PO_nIdUsuario` INT,
	IN `PO_nIdStatusDocumento` INT,
	IN `PO_sTipo` VARCHAR(10),
	IN `PO_sNombre` VARCHAR(80),
	IN `PO_sCalle` VARCHAR(150)



Validacion manual Licencia

CALL  sp_update_licencia_prospecto(5061,'2028-03-11', 2,'JOSE EDUARDO ARREDONDO SANDOVAL');

	IN `PO_nIdUsuario` INT,
	IN `PO_dFechaVigencia` VARCHAR(10),
	IN `PO_nIdStatusDocumento` INT,
	IN `PO_sNombre` VARCHAR(250)
