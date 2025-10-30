# 📊 Modelo Conceptual HÍBRIDO GESNOVA
## Nombres Semánticos ↔ Tablas Físicas
### Basado en Queries Reales de Producción

---

## 🎯 Objetivo de Este Documento

Este documento presenta el modelo de datos GESNOVA con **DOBLE NOMENCLATURA**:
- **Nombres Semánticos** (conceptos de negocio)
- **Nombres Físicos** (tablas reales de la base de datos)

Esto facilita:
✅ El entendimiento del negocio  
✅ La escritura de queries reales  
✅ La transición entre conceptos y código  

---

## 📋 Índice Rápido

1. [Tabla Maestra de Mapeo](#tabla-maestra-de-mapeo)
2. [Queries Reales Analizadas](#queries-reales-analizadas)
3. [Entidades Principales con Doble Nomenclatura](#entidades-principales)
4. [Casos de Uso con Código Real](#casos-de-uso-reales)
5. [Diccionario de Columnas](#diccionario-de-columnas)

---

## 🗺️ Tabla Maestra de Mapeo

### Vista Completa: Concepto → Tabla → Alias

| 💡 Concepto Semántico | 🔧 Tabla Física | 📝 Alias Común en Queries | 🎯 Uso Principal |
|----------------------|-----------------|---------------------------|------------------|
| **ORGANIZACION** | `ad_org` | `org` | Sociedad/Entidad legal (Ej: C035) |
| **INFO_ORGANIZACION** | `ad_orginfo` | `oi` | RUT de la organización |
| **PACIENTE** | `c_bpartner` | `bp`, `bp_paciente` | Persona que recibe atención |
| **ASEGURADORA** | `c_bpartner` | `bp_aseguradora` | Compañía de seguros |
| **DEVENGO_PRESTACION** | `i_accrual` | `iacc`, `ia` | ⭐ **TABLA CENTRAL** - Prestaciones realizadas |
| **EPISODIO_CLINICO** | (derivado de `i_accrual`) | - | Periodo de atención continua |
| **PRESTACION_MEDICA** | `m_product` | `prod`, `p` | Catálogo de servicios médicos |
| **CATEGORIA_PRESTACION** | `m_product_category` | - | Agrupación de prestaciones |
| **CENTRO_MEDICO** | `ex_locale` | `loc` | Lugar físico de atención (Ej: CBA) |
| **FACTURA** | `c_invoice` | `inv` | Documento de cobro |
| **LINEA_FACTURA** | `c_invoiceline` | `il` | Detalle de prestaciones facturadas |
| **TIPO_DOCUMENTO** | `c_doctype` | `dt` | Boleta/Factura/NC/Exenta |
| **TRANSACCION_CAJA** | `ex_postransaction` | `trx` | Registro de pago en POS |
| **LINEA_TRANSACCION** | `ex_postransactionline` | `trxline` | Detalle de medios de pago |
| **CAJA** | `ex_possession` | `pos` | Caja registradora física |
| **PUNTO_VENTA** | `c_pos` | `cpos` | Terminal de venta |
| **CAJERO** | `ad_user` | `usr` | Operador de caja |
| **TIPO_PAGO** | `ex_tendertype` | `tend` | Medio de pago (efectivo/tarjeta) |
| **PAGO** | `c_payment` | `pay` | Registro de pago a factura |
| **CUENTA_CONTABLE** | (referencia en `i_accrual`) | - | Cuenta mayor contable |

---

## 📑 Queries Reales Analizadas

### Query 1: Devengado (Cargos Devengados)
**Archivo:** `Devengado.sql`

#### 🎯 Propósito
Obtener todas las prestaciones devengadas (realizadas pero aún no necesariamente facturadas) en un periodo.

#### 🔧 Tablas Físicas Usadas
```sql
FROM atemplate.i_accrual AS iacc
LEFT JOIN atemplate.c_bpartner AS bp_paciente
LEFT JOIN atemplate.c_bpartner AS bp_aseguradora
```

#### 💡 Concepto de Negocio
```
DEVENGO_PRESTACION + PACIENTE + ASEGURADORA
```

#### 📊 Columnas Clave

| Columna en Query | Tabla Física | Campo Físico | Significado Semántico |
|-----------------|--------------|--------------|----------------------|
| `Fecha_Transaccion` | `i_accrual` | `datetrx` | Fecha registro del devengo |
| `Centro_Medico` | `i_accrual` | `ex_locale_name` | Nombre del centro |
| `Episodio` | `i_accrual` | `ex_episodevalue` | Código del episodio |
| `Nombre_Paciente` | `i_accrual` | `bpartnername` | Nombre (desnormalizado) |
| `RUT_Paciente` | `c_bpartner` | `taxid` | RUT del paciente |
| `Codigo_Prestacion` | `i_accrual` | `productvalue` | Código de servicio |
| `Prestacion` | `i_accrual` | `description` | Nombre de la prestación |
| `Cantidad` | `i_accrual` | `qtyordered` | Unidades prestadas |
| `Fecha_Prestacion_Real` | `i_accrual` | `datefrom` | Cuándo se realizó |
| `Anula_o_Cargo_Nuevo` | `i_accrual` | `ex_processtype` | Tipo de movimiento |
| `Monto_Bruto` | `i_accrual` | `priceactual` | Valor sin IVA |
| `Monto_IVA` | `i_accrual` | `taxamt` | Impuesto |
| `Monto_Neto` | `i_accrual` | `linenetamt` | Total |
| `Nombre_Aseguradora` | `c_bpartner` | `name` | Pagador |
| `Cuenta_Mayor` | `i_accrual` | `ex_mainaccount` | Cuenta contable |

#### 🔗 Joins Importantes
```sql
-- Paciente (join por código value)
LEFT JOIN c_bpartner AS bp_paciente 
    ON bp_paciente.value = iacc.bpartnervalue

-- Aseguradora (join por código value)
LEFT JOIN c_bpartner AS bp_aseguradora 
    ON bp_aseguradora.value = iacc.payorvalue
```

**⚠️ NOTA:** Los joins son por **VALUE** (código), no por ID directo.

---

### Query 2: Venta Online (Transacciones POS)
**Archivo:** `Venta_Online_2.sql`

#### 🎯 Propósito
Obtener todas las transacciones procesadas en punto de venta con contexto completo.

#### 🔧 Tablas Físicas Usadas
```sql
FROM atemplate.ex_postransaction AS trx
JOIN atemplate.ex_postransactionline AS trxline
JOIN atemplate.ex_possession AS pos
JOIN atemplate.ad_user AS usr
JOIN atemplate.c_bpartner AS bp
JOIN atemplate.c_pos AS cpos
JOIN atemplate.ex_locale AS loc
JOIN atemplate.ad_org AS org
```

#### 💡 Concepto de Negocio
```
TRANSACCION_CAJA + LINEA_TRANSACCION + CAJA + CAJERO + 
PUNTO_VENTA + CENTRO_MEDICO + PACIENTE + ORGANIZACION
```

#### 📊 Columnas Clave

| Columna en Query | Tabla Física | Campo Físico | Significado Semántico |
|-----------------|--------------|--------------|----------------------|
| `Numero_Transaccion_POS` | `ex_postransaction` | `value` | Número único de transacción |
| `Fecha_Proceso` | `ex_postransaction` | `created::date` | Día de la transacción |
| `Hora_Proceso` | `ex_postransaction` | `to_char(created, 'HH24:MI:SS')` | Hora exacta |
| `Nombre_Paciente` | `c_bpartner` | `name` | Cliente que paga |
| `RUT_Paciente` | `c_bpartner` | `taxid` | RUT del cliente |
| `Sociedad` | `ad_org` | `value` | Código org (Ej: C035) |
| `Centro` | `ex_locale` | `value` | Código centro (Ej: CBA) |
| `Usuario_Cajero` | `ad_user` | `name` | Nombre del cajero |
| `Nombre_Centro_Medico` | `ex_locale` | `name` | Nombre completo centro |
| `Monto_Neto` | `ex_postransactionline` | `linenetamt` | Monto de la línea |

#### 🔗 Joins en Cadena
```sql
ex_postransaction (trx)
    ↓ [ex_postransaction_id]
ex_postransactionline (trxline)

ex_postransaction (trx)
    ↓ [ex_possession_id]
ex_possession (pos)
    ↓ [c_pos_id]
c_pos (cpos)
    ↓ [ex_locale_id]
ex_locale (loc)

ex_possession (pos)
    ↓ [createdby]
ad_user (usr)

ex_postransaction (trx)
    ↓ [c_bpartner_id]
c_bpartner (bp)
```

**💡 INSIGHT:** El cajero se obtiene desde `ex_possession.createdby`, no desde la transacción directamente.

---

### Query 3: Recaudación (Cobros con Medio de Pago)
**Archivo:** `Recaudacion.sql`

#### 🎯 Propósito
Obtener el detalle de recaudación por medio de pago (efectivo, tarjeta, etc.).

#### 🔧 Tablas Físicas Usadas
```sql
FROM atemplate.ex_postransaction AS trx
JOIN atemplate.ex_postransactionline AS trxline
LEFT JOIN atemplate.c_payment AS pay
LEFT JOIN atemplate.ex_tendertype AS tend
LEFT JOIN atemplate.c_invoice AS inv
LEFT JOIN atemplate.c_doctype AS dt
JOIN atemplate.ex_possession AS pos
JOIN atemplate.ad_user AS usr
JOIN atemplate.c_bpartner AS bp
JOIN atemplate.c_pos AS cpos
JOIN atemplate.ex_locale AS loc
```

#### 💡 Concepto de Negocio
```
TRANSACCION_CAJA + LINEA_TRANSACCION + PAGO + TIPO_PAGO + 
FACTURA + TIPO_DOCUMENTO + CAJA + CAJERO + CENTRO_MEDICO + PACIENTE
```

#### 📊 Columnas Clave

| Columna en Query | Tabla Física | Campo Físico | Significado Semántico |
|-----------------|--------------|--------------|----------------------|
| `Numero_Transaccion_POS` | `ex_postransaction` | `value` | ID de transacción |
| `Fecha_Hora_Transaccion` | `ex_postransaction` | `created` | Timestamp completo |
| `Nombre_Paciente` | `c_bpartner` | `name` | Cliente |
| `RUT_Paciente` | `c_bpartner` | `taxid` | RUT |
| `Usuario_Cajero` | `ad_user` | `name` | Cajero |
| `N° Caja` | `ex_possession` | `ex_possession_id` | ID de caja |
| `Nombre_Centro_Medico` | `ex_locale` | `name` | Centro |
| `Nombre_Medio_de_Pago` | `ex_tendertype` / `c_doctype` | `name` / `printname` | Medio de pago |
| `Monto_Aplicado` | `ex_postransactionline` | `linenetamt` | Monto pagado |

#### 🔗 Joins Complejos
```sql
-- La línea puede vincular a un PAGO
ex_postransactionline (trxline)
    ↓ [c_payment_id] (LEFT JOIN)
c_payment (pay)
    ↓ [ex_tendertype_id]
ex_tendertype (tend)

-- O puede vincular a una FACTURA
ex_postransactionline (trxline)
    ↓ [c_invoice_id] (LEFT JOIN)
c_invoice (inv)
    ↓ [c_doctype_id]
c_doctype (dt)

-- Lógica de nombre medio de pago:
COALESCE(tend.name, dt.printname)
```

**💡 INSIGHT:** El medio de pago se obtiene de dos fuentes alternativas: `ex_tendertype` si hay pago directo, o `c_doctype` si la línea referencia una factura.

---

### Query 4: Venta IntegraMédica (Facturación Detallada)
**Archivo:** `Venta_Integramedica.sql`

#### 🎯 Propósito
Obtener el detalle de facturación con desglose de coberturas (seguro, copago, excedentes).

#### 🔧 Tablas Físicas Usadas
```sql
FROM atemplate.c_invoiceline il
JOIN atemplate.c_invoice inv
JOIN atemplate.c_doctype dt
JOIN atemplate.c_bpartner bp
JOIN atemplate.m_product prod
JOIN atemplate.c_pos cpos
JOIN atemplate.ex_locale loc
JOIN atemplate.ad_orginfo oi
```

#### 💡 Concepto de Negocio
```
FACTURA + LINEA_FACTURA + TIPO_DOCUMENTO + PACIENTE + 
PRESTACION_MEDICA + PUNTO_VENTA + CENTRO_MEDICO + INFO_ORGANIZACION
```

#### 📊 Columnas Clave

| Columna en Query | Tabla Física | Campo Físico | Significado Semántico |
|-----------------|--------------|--------------|----------------------|
| `Numero_Factura` | `c_invoice` | `documentno` | Número del documento |
| `Tipo_Documento` | `c_doctype` | `printname` | Boleta/Factura/NC |
| `Fecha_Venta` | `c_invoice` | `dateinvoiced` | Fecha de emisión |
| `Nombre_Paciente` | `c_bpartner` | `name` | Cliente facturado |
| `RUT_Paciente` | `c_bpartner` | `taxid` | RUT |
| `Nombre_Prestacion` | `m_product` | `name` | Servicio facturado |
| `Monto_Total_Linea` | `c_invoiceline` | `linenetamt` | Total línea |
| `Monto_Seguro_Comp` | `c_invoiceline` | `ex_insuranceamount` | 💰 Paga el seguro |
| `Monto_Copago` | `c_invoiceline` | `ex_copayment` | 💳 Paga el paciente |
| `Monto_Excedentes` | `c_invoiceline` | `ex_surplusamount` | No cubierto |
| `Centro_Medico` | `ex_locale` | `name` | Lugar de emisión |
| `RUT_Organizacion` | `ad_orginfo` | `taxid` | RUT de la sociedad |

#### 🧮 Validación Financiera CRÍTICA
```sql
-- Debe cumplirse SIEMPRE:
Monto_Total_Linea = Monto_Seguro_Comp + Monto_Copago + Monto_Excedentes

-- En SQL:
il.linenetamt = il.ex_insuranceamount + il.ex_copayment + il.ex_surplusamount
```

#### 📋 Filtro de Tipos de Documento
```sql
WHERE dt.printname IN (
    'Boleta',
    'Boleta Manual',
    'Exempt Ticket',         -- Boleta Exenta
    'Invoice',               -- Factura
    'Factura Exenta',        
    'Credit Memo',           -- Nota de crédito
    'Nota de Credito (test)'
)
```

---

## 🏗️ Entidades Principales con Doble Nomenclatura

### 1. DEVENGO_PRESTACION ⭐ (i_accrual)
**Tabla Central del Sistema**

#### Nombres de Columnas: Físico → Semántico

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `i_accrual_id` | id_devengo | Llave primaria | 12345 |
| `ad_org_id` | id_organizacion | FK a organización | 1000 |
| `datetrx` | fecha_transaccion | Fecha registro | 2025-10-15 |
| `ex_episodevalue` | codigo_episodio | Código del episodio | EP-2025-001 |
| `bpartnervalue` | codigo_paciente | Código del paciente | PAC-12345 |
| `bpartnername` | nombre_paciente | Nombre (desnormalizado) | Juan Pérez |
| `productvalue` | codigo_prestacion | Código del servicio | PREST-001 |
| `description` | descripcion_prestacion | Nombre del servicio | Consulta Médica |
| `datefrom` | fecha_prestacion_real | Cuándo se realizó | 2025-10-15 |
| `dateto` | fecha_hasta | Fin del servicio | 2025-10-15 |
| `qtyordered` | cantidad | Unidades | 1 |
| `priceactual` | monto_bruto | Sin IVA | 10000 |
| `taxamt` | monto_iva | Impuesto | 1900 |
| `linenetamt` | monto_neto | Total | 11900 |
| `orgvalue` | codigo_organizacion | Código sociedad | C035 |
| `ex_locale_name` | centro_medico | Nombre del centro | Clínica Las Condes |
| `payorvalue` | codigo_aseguradora | Código pagador | ISAPRE-001 |
| `payor_name` | nombre_aseguradora | Nombre pagador | Isapre Banmédica |
| `ex_mainaccount` | cuenta_mayor | Cuenta contable | 410101 |
| `ex_accountname` | nombre_cuenta | Descripción cuenta | Ingresos Consultas |
| `ex_processtype` | tipo_proceso | Cargo/Anulación | NEW/CANCEL |
| `processed` | procesado | Estado | Y/N |

#### Relaciones Clave
```sql
-- Join a PACIENTE
i_accrual.bpartnervalue = c_bpartner.value (WHERE iscustomer='Y')

-- Join a ASEGURADORA
i_accrual.payorvalue = c_bpartner.value (rol payor)

-- Join a PRESTACION_MEDICA
i_accrual.productvalue = m_product.value

-- Episodio (agrupación lógica)
GROUP BY i_accrual.ex_episodevalue
```

---

### 2. FACTURA (c_invoice) + LINEA_FACTURA (c_invoiceline)

#### FACTURA - Nombres de Columnas

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `c_invoice_id` | id_factura | Llave primaria | 56789 |
| `documentno` | numero_factura | Número documento | FAC-001234 |
| `dateinvoiced` | fecha_venta / fecha_emision | Cuándo se emitió | 2025-10-20 |
| `c_bpartner_id` | id_paciente | FK a cliente | 9876 |
| `c_doctype_id` | id_tipo_documento | FK a tipo | 5 |
| `c_pos_id` | id_punto_venta | FK a POS | 10 |
| `ex_episode_id` | id_episodio | FK a episodio | 123 |
| `ex_episode_value` | codigo_episodio | Código episodio | EP-2025-001 |
| `grandtotal` | monto_total | Total factura | 50000 |
| `totallines` | subtotal_lineas | Subtotal | 45000 |
| `docstatus` | estado_documento | CO/DR/VO | CO |
| `ispaid` | pagado | Está pagada | Y/N |
| `duedate` | fecha_vencimiento | Vence | 2025-11-20 |

#### LINEA_FACTURA - Nombres de Columnas

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `c_invoiceline_id` | id_linea_factura | Llave primaria | 67890 |
| `c_invoice_id` | id_factura | FK a factura | 56789 |
| `m_product_id` | id_prestacion | FK a prestación | 111 |
| `qtyinvoiced` | cantidad_facturada | Unidades | 1 |
| `priceactual` | precio_unitario | Precio unitario | 10000 |
| `linenetamt` | monto_total_linea | **Total** | 10000 |
| `ex_insuranceamount` | monto_seguro_comp | 🏥 Paga seguro | 8000 |
| `ex_copayment` | monto_copago | 💳 Paga paciente | 1500 |
| `ex_surplusamount` | monto_excedentes | No cubierto | 500 |
| `description` | descripcion | Descripción | Consulta Médica |

**💰 Fórmula CRÍTICA:**
```
linenetamt = ex_insuranceamount + ex_copayment + ex_surplusamount
```

---

### 3. TRANSACCION_CAJA (ex_postransaction) + LINEA_TRANSACCION (ex_postransactionline)

#### TRANSACCION_CAJA - Nombres de Columnas

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `ex_postransaction_id` | id_transaccion | Llave primaria | 99999 |
| `value` | numero_transaccion_pos | Número único | TRX-2025-12345 |
| `created` | fecha_hora_transaccion | Timestamp | 2025-10-20 14:35:22 |
| `created::date` | fecha_proceso | Solo fecha | 2025-10-20 |
| `HH24:MI:SS` | hora_proceso | Solo hora | 14:35:22 |
| `ex_possession_id` | id_caja | FK a caja | 5 |
| `c_bpartner_id` | id_paciente | FK a cliente | 9876 |
| `ad_user_id` | id_cajero | FK a usuario | 22 |
| `docstatus` | estado_documento | CO/DR | CO |

#### LINEA_TRANSACCION - Nombres de Columnas

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `ex_postransactionline_id` | id_linea_transaccion | Llave primaria | 88888 |
| `ex_postransaction_id` | id_transaccion | FK a transacción | 99999 |
| `c_payment_id` | id_pago | FK a pago (opt) | 555 |
| `c_invoice_id` | id_factura | FK a factura (opt) | 56789 |
| `linenetamt` | monto_aplicado / monto_neto | Monto | 10000 |

---

### 4. PACIENTE y ASEGURADORA (c_bpartner)

#### Estructura Compartida

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Para PACIENTE | 📝 Para ASEGURADORA |
|-----------------|-------------------|-----------------|-------------------|
| `c_bpartner_id` | id | ID único | ID único |
| `value` | codigo | codigo_paciente | codigo_aseguradora |
| `taxid` | rut | rut_paciente | rut_aseguradora |
| `name` | nombre | nombre_completo | nombre_aseguradora |
| `name2` | apellidos / comercial | apellidos | nombre_comercial |
| `iscustomer` | es_cliente | Y | N |
| `isvendor` | es_proveedor | N | Y (a veces) |
| `ex_payor_id` | id_aseguradora | FK a aseguradora | NULL |
| `birthday` | fecha_nacimiento | fecha | NULL |
| `gender` | genero | M/F | NULL |
| `bloodgroup` | grupo_sanguineo | O+/A+/etc | NULL |
| `phone` | telefono | teléfono | teléfono |
| `email` | email | email | email |

#### Cómo Distinguirlos
```sql
-- PACIENTE
SELECT * FROM c_bpartner WHERE iscustomer = 'Y'

-- ASEGURADORA
SELECT * FROM c_bpartner 
WHERE value IN (SELECT DISTINCT payorvalue FROM i_accrual WHERE payorvalue IS NOT NULL)
   OR c_bpartner_id IN (SELECT DISTINCT ex_payor_id FROM c_bpartner WHERE ex_payor_id IS NOT NULL)
```

---

### 5. CENTRO_MEDICO (ex_locale) + PUNTO_VENTA (c_pos) + CAJA (ex_possession)

#### CENTRO_MEDICO

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `ex_locale_id` | id_centro | ID único | 7 |
| `value` | codigo_centro | Código corto | CBA |
| `name` | nombre_centro_medico | Nombre completo | Clínica Las Condes |
| `c_location_id` | id_ubicacion | FK a dirección | 100 |

#### PUNTO_VENTA

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `c_pos_id` | id_punto_venta | ID único | 10 |
| `value` | codigo_pos | Código | POS-CBA-01 |
| `name` | nombre_pos | Nombre | POS Clínica BA |
| `ex_locale_id` | id_centro_medico | FK a centro | 7 |

#### CAJA

| 🔧 Nombre Físico | 💡 Nombre Semántico | 📝 Descripción | 🎯 Ejemplo |
|-----------------|-------------------|---------------|-----------|
| `ex_possession_id` | id_caja | ID único | 5 |
| `value` | codigo_caja | Código | CAJA-01 |
| `name` | nombre_caja | Nombre | Caja Principal CBA |
| `c_pos_id` | id_punto_venta | FK a POS | 10 |
| `createdby` | id_usuario_creador | Usuario que la abrió | 22 |

#### Relación
```
CENTRO_MEDICO (ex_locale)
    ↓
PUNTO_VENTA (c_pos)
    ↓
CAJA (ex_possession)
```

---

## 🎯 Casos de Uso Reales

### Caso 1: Obtener Devengos de un Episodio

**Concepto:**
```
Buscar todas las PRESTACIONES devengadas de un EPISODIO específico
```

**Query con Nombres Híbridos:**
```sql
-- Concepto: DEVENGO_PRESTACION + PACIENTE + PRESTACION_MEDICA
-- Físico:   i_accrual + c_bpartner + m_product

SELECT 
    -- Episodio
    iacc.ex_episodevalue AS codigo_episodio,
    
    -- Paciente
    iacc.bpartnername AS nombre_paciente,
    bp.taxid AS rut_paciente,
    
    -- Prestación
    iacc.productvalue AS codigo_prestacion,
    iacc.description AS descripcion_prestacion,
    prod.name AS nombre_prestacion_completo,
    
    -- Fechas
    iacc.datefrom AS fecha_prestacion_real,
    
    -- Montos
    iacc.priceactual AS monto_bruto,
    iacc.taxamt AS monto_iva,
    iacc.linenetamt AS monto_neto,
    
    -- Centro Médico
    iacc.ex_locale_name AS centro_medico,
    
    -- Aseguradora
    iacc.payorvalue AS codigo_aseguradora,
    iacc.payor_name AS nombre_aseguradora

FROM atemplate.i_accrual AS iacc
LEFT JOIN atemplate.c_bpartner AS bp 
    ON bp.value = iacc.bpartnervalue
LEFT JOIN atemplate.m_product AS prod 
    ON prod.value = iacc.productvalue
    
WHERE iacc.ex_episodevalue = 'EP-2025-001'
ORDER BY iacc.datefrom;
```

---

### Caso 2: Facturación del Día con Desglose

**Concepto:**
```
Obtener todas las FACTURAS del día con desglose de COBERTURA
```

**Query con Nombres Híbridos:**
```sql
-- Concepto: FACTURA + LINEA_FACTURA + PACIENTE + PRESTACION_MEDICA + TIPO_DOCUMENTO
-- Físico:   c_invoice + c_invoiceline + c_bpartner + m_product + c_doctype

SELECT 
    -- Factura
    inv.documentno AS numero_factura,
    dt.printname AS tipo_documento,
    inv.dateinvoiced AS fecha_venta,
    
    -- Paciente
    bp.name AS nombre_paciente,
    bp.taxid AS rut_paciente,
    
    -- Prestación
    prod.name AS nombre_prestacion,
    il.qtyinvoiced AS cantidad,
    il.priceactual AS precio_unitario,
    
    -- DESGLOSE FINANCIERO
    il.linenetamt AS monto_total_linea,
    il.ex_insuranceamount AS monto_seguro_comp,
    il.ex_copayment AS monto_copago,
    il.ex_surplusamount AS monto_excedentes,
    
    -- Verificación
    (il.ex_insuranceamount + il.ex_copayment + il.ex_surplusamount) AS suma_verificacion,
    
    -- Centro Médico
    loc.name AS centro_medico,
    
    -- Organización
    oi.taxid AS rut_organizacion

FROM atemplate.c_invoiceline il
INNER JOIN atemplate.c_invoice inv 
    ON il.c_invoice_id = inv.c_invoice_id
INNER JOIN atemplate.c_doctype dt 
    ON inv.c_doctype_id = dt.c_doctype_id
INNER JOIN atemplate.c_bpartner bp 
    ON inv.c_bpartner_id = bp.c_bpartner_id
INNER JOIN atemplate.m_product prod 
    ON il.m_product_id = prod.m_product_id
INNER JOIN atemplate.c_pos cpos 
    ON inv.c_pos_id = cpos.c_pos_id
INNER JOIN atemplate.ex_locale loc 
    ON cpos.ex_locale_id = loc.ex_locale_id
INNER JOIN atemplate.ad_orginfo oi 
    ON inv.ad_org_id = oi.ad_org_id

WHERE inv.dateinvoiced = CURRENT_DATE
    AND inv.docstatus = 'CO'  -- Solo completas
    
ORDER BY inv.documentno;
```

---

### Caso 3: Cierre de Caja por Medio de Pago

**Concepto:**
```
Obtener RECAUDACION de una CAJA agrupada por MEDIO_DE_PAGO
```

**Query con Nombres Híbridos:**
```sql
-- Concepto: TRANSACCION_CAJA + LINEA_TRANSACCION + TIPO_PAGO + CAJA + CAJERO
-- Físico:   ex_postransaction + ex_postransactionline + ex_tendertype + ex_possession + ad_user

SELECT 
    -- Caja
    pos.name AS nombre_caja,
    pos.ex_possession_id AS numero_caja,
    
    -- Cajero
    usr.name AS nombre_cajero,
    
    -- Centro Médico
    loc.name AS nombre_centro_medico,
    
    -- Medio de Pago
    COALESCE(tend.name, dt.printname) AS medio_de_pago,
    
    -- Estadísticas
    COUNT(DISTINCT trx.ex_postransaction_id) AS cantidad_transacciones,
    COUNT(*) AS cantidad_lineas,
    SUM(trxline.linenetamt) AS total_recaudado

FROM atemplate.ex_postransaction trx
INNER JOIN atemplate.ex_postransactionline trxline 
    ON trxline.ex_postransaction_id = trx.ex_postransaction_id
    
LEFT JOIN atemplate.c_payment pay 
    ON pay.c_payment_id = trxline.c_payment_id
LEFT JOIN atemplate.ex_tendertype tend 
    ON tend.ex_tendertype_id = pay.ex_tendertype_id
    
LEFT JOIN atemplate.c_invoice inv 
    ON inv.c_invoice_id = trxline.c_invoice_id
LEFT JOIN atemplate.c_doctype dt 
    ON dt.c_doctype_id = inv.c_doctype_id
    
INNER JOIN atemplate.ex_possession pos 
    ON pos.ex_possession_id = trx.ex_possession_id
INNER JOIN atemplate.ad_user usr 
    ON usr.ad_user_id = pos.createdby
    
INNER JOIN atemplate.c_pos cpos 
    ON cpos.c_pos_id = pos.c_pos_id
INNER JOIN atemplate.ex_locale loc 
    ON loc.ex_locale_id = cpos.ex_locale_id

WHERE DATE(trx.created) = CURRENT_DATE
    AND pos.name = 'Caja Principal'
    AND trx.docstatus = 'CO'

GROUP BY 
    pos.name,
    pos.ex_possession_id,
    usr.name,
    loc.name,
    COALESCE(tend.name, dt.printname)
    
ORDER BY total_recaudado DESC;
```

---

## 📚 Diccionario Completo de Columnas

### Columnas Más Usadas

| 🔧 Nombre Físico | 💡 Alias/Nombre Semántico | 🏢 Tabla(s) | 📝 Descripción | 🎯 Ejemplo |
|-----------------|--------------------------|------------|---------------|-----------|
| `value` | codigo_* | Múltiples | Código alfanumérico único | C035, PAC-123, CBA |
| `name` | nombre_* | Múltiples | Nombre descriptivo | Juan Pérez, Clínica BA |
| `taxid` | rut_* | c_bpartner, ad_orginfo | RUT chileno | 12345678-9 |
| `created` | fecha_hora_* | Múltiples | Timestamp de creación | 2025-10-20 14:35:22 |
| `datetrx` | fecha_transaccion | i_accrual | Fecha de registro | 2025-10-20 |
| `datefrom` | fecha_desde / fecha_prestacion_real | i_accrual | Fecha real del servicio | 2025-10-20 |
| `dateinvoiced` | fecha_venta / fecha_emision | c_invoice | Fecha emisión factura | 2025-10-20 |
| `documentno` | numero_factura / numero_documento | c_invoice | Número correlativo | FAC-001234 |
| `linenetamt` | monto_neto / monto_total_linea | i_accrual, c_invoiceline, ex_postransactionline | Monto total | 10000 |
| `priceactual` | monto_bruto / precio_unitario | i_accrual, c_invoiceline | Precio sin IVA | 8403 |
| `taxamt` | monto_iva | i_accrual | Impuesto | 1597 |
| `qtyordered` | cantidad | i_accrual | Cantidad ordenada | 1 |
| `qtyinvoiced` | cantidad_facturada | c_invoiceline | Cantidad en factura | 1 |
| `ex_insuranceamount` | monto_seguro_comp | c_invoiceline | Cobertura seguro | 8000 |
| `ex_copayment` | monto_copago | c_invoiceline | Copago paciente | 1500 |
| `ex_surplusamount` | monto_excedentes | c_invoiceline | No cubierto | 500 |
| `ex_episodevalue` | codigo_episodio | i_accrual | Código episodio | EP-2025-001 |
| `bpartnervalue` | codigo_paciente | i_accrual | Código paciente | PAC-123 |
| `bpartnername` | nombre_paciente | i_accrual | Nombre (desnorm) | Juan Pérez |
| `productvalue` | codigo_prestacion | i_accrual | Código prestación | PREST-001 |
| `description` | descripcion_* | Múltiples | Descripción texto | Consulta Médica |
| `payorvalue` | codigo_aseguradora | i_accrual | Código pagador | ISAPRE-001 |
| `payor_name` | nombre_aseguradora | i_accrual | Nombre pagador | Banmédica |
| `ex_locale_name` | centro_medico | i_accrual | Nombre centro | Clínica BA |
| `ex_mainaccount` | cuenta_mayor | i_accrual | Cuenta contable | 410101 |
| `orgvalue` | codigo_organizacion | i_accrual | Código sociedad | C035 |
| `ex_processtype` | tipo_proceso / anula_o_cargo_nuevo | i_accrual | Tipo movimiento | NEW/CANCEL |
| `docstatus` | estado_documento | c_invoice, ex_postransaction | Estado | CO/DR/VO |
| `ispaid` | pagado | c_invoice | Factura pagada | Y/N |
| `iscustomer` | es_cliente | c_bpartner | Es paciente | Y/N |
| `printname` | tipo_documento | c_doctype | Tipo documento | Boleta/Factura |

---

## 🎓 Patrones de Join Importantes

### Patrón 1: Join por VALUE (código)
```sql
-- COMÚN en i_accrual
i_accrual.bpartnervalue = c_bpartner.value
i_accrual.payorvalue = c_bpartner.value
i_accrual.productvalue = m_product.value
```
**⚠️ NOTA:** No se usa FK directo, sino código alfanumérico.

### Patrón 2: Join por ID (relación directa)
```sql
-- COMÚN en facturas
c_invoiceline.c_invoice_id = c_invoice.c_invoice_id
c_invoice.c_bpartner_id = c_bpartner.c_bpartner_id
c_invoice.m_product_id = m_product.m_product_id
```

### Patrón 3: Join en Cadena (POS)
```sql
-- Para llegar de TRANSACCION a CENTRO_MEDICO
ex_postransaction 
    → ex_possession 
    → c_pos 
    → ex_locale
```

### Patrón 4: COALESCE para Alternativas
```sql
-- Nombre medio de pago de dos fuentes
COALESCE(ex_tendertype.name, c_doctype.printname)
```

---

## 💡 Insights Clave

### 1. Desnormalización en i_accrual
La tabla `i_accrual` contiene campos desnormalizados:
- `bpartnername` (ya está el nombre del paciente)
- `payor_name` (nombre de la aseguradora)
- `ex_locale_name` (nombre del centro)

**Ventaja:** Queries más rápidas  
**Desventaja:** Debe mantenerse sincronizado

### 2. Estados de Documento
```sql
docstatus:
    'DR' = Draft (Borrador)
    'CO' = Completed (Completado)
    'VO' = Voided (Anulado)
    'CL' = Closed (Cerrado)
```

### 3. Tipos de Documento Comunes
```sql
printname:
    'Boleta'
    'Boleta Manual'
    'Exempt Ticket' (Boleta Exenta)
    'Invoice' (Factura)
    'Factura Exenta'
    'Credit Memo' (Nota de Crédito)
```

### 4. Campos con Prefijo ex_
Campos que empiezan con `ex_` son **extensiones** específicas de GESNOVA sobre el modelo base (probablemente iDempiere/ADempiere):
- `ex_episodevalue`
- `ex_locale_name`
- `ex_insuranceamount`
- `ex_copayment`
- `ex_surplusamount`
- `ex_mainaccount`
- `ex_processtype`

---

## 📊 Resumen Visual

```
FLUJO DE NEGOCIO:

1. DEVENGO (i_accrual)
   ↓
   Prestación realizada a PACIENTE (c_bpartner)
   en CENTRO_MEDICO (ex_locale)
   dentro de EPISODIO (ex_episodevalue)
   cubierta por ASEGURADORA (c_bpartner.payorvalue)
   
2. FACTURACION (c_invoice + c_invoiceline)
   ↓
   Factura emitida desde PUNTO_VENTA (c_pos)
   con TIPO_DOCUMENTO (c_doctype)
   desglosando SEGURO + COPAGO + EXCEDENTES
   
3. COBRO (ex_postransaction + ex_postransactionline)
   ↓
   Transacción en CAJA (ex_possession)
   operada por CAJERO (ad_user)
   con MEDIO_DE_PAGO (ex_tendertype)
   aplicando a FACTURA (c_invoice)
```

---

## 🚀 Recomendaciones Finales

### Para Desarrolladores
1. **Siempre usar aliases claros** en queries:
   - `iacc` para `i_accrual`
   - `bp` para `c_bpartner`
   - `inv` para `c_invoice`
   - `trx` para `ex_postransaction`

2. **Documentar joins por VALUE**:
   ```sql
   -- Join por código, no por ID
   LEFT JOIN c_bpartner AS bp_paciente 
       ON bp_paciente.value = iacc.bpartnervalue
   ```

3. **Validar sumas financieras**:
   ```sql
   -- Siempre verificar
   WHERE ABS(
       il.linenetamt - 
       (il.ex_insuranceamount + il.ex_copayment + il.ex_surplusamount)
   ) < 0.01
   ```

### Para Analistas
1. Usar los **nombres semánticos** en reportes
2. Referenciar **tabla física** en documentación técnica
3. Mantener **ambas nomenclaturas** en queries complejas

### Para el Equipo
1. Actualizar este documento cuando cambien las queries
2. Agregar nuevos patrones de join descubiertos
3. Documentar extensiones (campos `ex_*`) nuevas

---

**Fin del Documento**

*Última actualización: basado en queries reales de Octubre 2025*  
*Archivos analizados: Devengado.sql, Venta_Online_2.sql, Recaudacion.sql, Venta_Integramedica.sql*
