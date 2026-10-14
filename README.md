# RS_Recaudacion – Reportería de recaudación en Power BI Report Server

Suite de **10 reportes paginados (.rdl)** para el control diario de la recaudación de una
entidad pública: cobros por ventanilla, banca y canales electrónicos, convenios de pago,
transferencias SPI, códigos electrónicos de pago (CEP) y reversos.
Publicados en **Power BI Report Server** sobre un **data mart de recaudación en SQL Server**.

> Versión saneada de reportes en producción. Se reemplazaron los servidores por marcadores
> (`SERVIDOR_DW`), se retiró la identidad institucional y no se incluye ningún dato:
> los `.rdl` contienen solo la definición del reporte (consultas, parámetros y diseño).

## Arquitectura

```text
Sistema de recaudación (OLTP)
        │  ETL
        ▼
RECAUDACION_DM (SQL Server)                      CONVENIOS_DM
  esquema DesgloseCobros                           esquema ConveniosPago
  ├── FactCobro            ── cabecera del cobro     ├── FactDetalleValoresConvenio
  ├── FactDesgloseCobro    ── rubros del cobro       └── FactTablaAmortizacion
  ├── FactCobrosConTransaferenciaSpi
  ├── DimTransaccion / DimTransaccionDesglose
  ├── DimContribuyente · DimBanco · DimCanalDePago
  └── DimFechaEntidadExterna
        │
        ▼
Power BI Report Server  ──  10 reportes .rdl parametrizados
```

## Reportes

| # | Reporte | Para qué sirve | Parámetros principales |
|---|---|---|---|
| 01 | Listado de Recaudaciones – Mercados | Cobros de mercados municipales con su desglose por rubro | Fechas, transacción, subtransacción, detalle de desglose |
| 02 | Recaudaciones VIA – No Tributario | Separa la recaudación no tributaria por tipo de ingreso | Fechas, transacción, subtransacción, tipo |
| 03 | Listado General de Recaudaciones | Listado maestro de cobros por transacción, canal y cajero | Fechas, transacción, subtransacción |
| 04 | Reporte General Origen Recaudación | Traza cada cobro de convenio a la deuda que lo originó (tabla de amortización) | Fechas, transacción, subtransacción |
| 05 | Reporte General SPI | Concilia cobros contra transferencias del Sistema de Pagos Interbancarios | Fechas de cobro y de transferencia, secuencia, valor, RUC |
| 06 | Reporte Origen Recaudación CIP-PDP | Origen de convenios de pago (CIP) y abonos provisionales (PDP) | Fechas, transacción, subtransacción |
| 07 | CEP | Consulta de códigos electrónicos de pago por contribuyente o cobro | CEP, id de cobro, identificación, nombre |
| 08 | CEP Detalle | Ficha de un CEP con sus cobros electrónicos asociados | CEP |
| 09 | Desglose Transacción | Recaudación abierta por transacción y por rubro de desglose | Fechas, transacción y desglose en cascada |
| 10 | Desglose Reversos | Control de reversos: qué se reversó, cuándo y sobre qué rubro | Fechas de cobro y de reverso, transacción, desglose |

## Técnicas aplicadas

- **Parámetros en cascada**: la subtransacción depende de la transacción elegida y el desglose depende de ambas.
  Todos aceptan el valor vacío (`''`), que significa "todos".
- **Fecha de actualización del data mart** en la cabecera de cada reporte, para que el usuario sepa hasta cuándo llegan los datos.
- **Rendimiento**: `WITH (NOLOCK)` en lecturas de hechos, `TOP` de seguridad en listados masivos y filtros por rango de fechas aplicados en el origen.
- **Modelo dimensional**: los reportes leen hechos y dimensiones del data mart, no el sistema transaccional.
- **Conciliación**: los reportes 04 y 06 cruzan cobros con la tabla de amortización de convenios; el 05 cruza cobros con transferencias SPI.

## Cómo abrirlos

1. Abrir el `.rdl` en **Power BI Report Builder** o en Visual Studio (Reporting Services).
2. Reemplazar `SERVIDOR_DW` en el origen de datos por el servidor que corresponda.
3. Publicar en Power BI Report Server o ejecutar en vista previa.

## Stack

SQL Server (T-SQL) · Data mart dimensional · Power BI Report Server · Report Builder (RDL)
