Reporte de Conciliación y Reversos
Este documento contiene las consultas necesarias para identificar discrepancias en operaciones conciliadas que presentan reversos.

1. Operaciones de Pagarés con Reversos
Objetivo: Identificar pagarés conciliados cuyos pagos asociados han sido marcados como refunded (reembolsados).

<pre>
/* Query para saber si tenemos ops de pagarés conciliados con pagos que tienen reversos */

SELECT
    dpad.nIdPagareAutomaticDetalle,
    drppa.nIdPagoPaycash,
    CONCAT(drppa.nIdPagareAutomaticDetalle, ",", drppa.nIdPagoPaycash) AS sPagoOp,
    hp.sIdPayment,
    JSON_EXTRACT(hp.`sPaymentData`, "$.refunded") AS sRefunded
FROM db_besta_management_prod.dat_pagare_automatic_detalle dpad
JOIN db_besta_management_prod.dat_relacion_pagos_pagare_automatic drppa
    ON drppa.nIdPagareAutomaticDetalle = dpad.nIdPagareAutomaticDetalle
JOIN db_besta_management_prod.dat_pagos_paycash dpp
    ON dpp.nIdPagoPaycash = drppa.nIdPagoPaycash
JOIN db_besta_management_prod.hst_payments hp
    ON hp.sIdPayment = dpp.nPaymentId;

</pre>

[!IMPORTANT] Acción requerida: Descargar el resultado, filtrar la columna sRefunded. Si el valor es true, existe una inconsistencia que debe ser reportada.

2. Operaciones de Semana con Reversos
Objetivo: Monitorear reversos en operaciones semanales. Correr este reporte desde el día 1 del mes en curso.

<pre>
/* Query para saber si tenemos ops de semana conciliados con pagos que tienen reversos */
SELECT
    oop.nIdOpsOP,
    drpo.nIdPagoPaycash,
    CONCAT(drpo.nIdOrdenPago, ",", drpo.nIdPagoPaycash) AS sPagoOp,
    hp.sIdPayment,
    JSON_EXTRACT(hp.`sPaymentData`, "$.refunded") AS sRefunded
FROM db_besta_management_prod.ops_orden_pago oop
JOIN db_besta_management_prod.dat_relacion_pagos_op drpo
    ON drpo.nIdOrdenPago = oop.nIdOpsOP
JOIN db_besta_management_prod.dat_pagos_paycash dpp
    ON dpp.nIdPagoPaycash = drpo.nIdPagoPaycash
JOIN db_besta_management_prod.hst_payments hp
    ON hp.sIdPayment = dpp.nPaymentId;
</pre>



Notas de Operación
Frecuencia: Se recomienda ejecutar estos queries al inicio de cada ciclo de pago.

