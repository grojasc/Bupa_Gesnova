# 🚀 CHEAT SHEET: GESNOVA - Nombres Semánticos ↔ Físicos

## 📋 Referencia Rápida Para Developers

---

## 🎯 Top 10 Tablas Más Usadas

| # | 💡 Nombre Semántico | 🔧 Tabla Física | 🏷️ Alias | ⭐ Importancia |
|---|-------------------|----------------|----------|---------------|
| 1 | **DEVENGO_PRESTACION** | `i_accrual` | `iacc`, `ia` | ⭐⭐⭐⭐⭐ CENTRAL |
| 2 | **PACIENTE** | `c_bpartner` | `bp`, `bp_paciente` | ⭐⭐⭐⭐⭐ |
| 3 | **FACTURA** | `c_invoice` | `inv` | ⭐⭐⭐⭐⭐ |
| 4 | **LINEA_FACTURA** | `c_invoiceline` | `il` | ⭐⭐⭐⭐⭐ |
| 5 | **TRANSACCION_CAJA** | `ex_postransaction` | `trx` | ⭐⭐⭐⭐ |
| 6 | **CENTRO_MEDICO** | `ex_locale` | `loc` | ⭐⭐⭐⭐ |
| 7 | **PRESTACION_MEDICA** | `m_product` | `prod`, `p` | ⭐⭐⭐⭐ |
| 8 | **ASEGURADORA** | `c_bpartner` | `bp_aseguradora` | ⭐⭐⭐⭐ |
| 9 | **ORGANIZACION** | `ad_org` | `org` | ⭐⭐⭐ |
| 10 | **TIPO_DOCUMENTO** | `c_doctype` | `dt` | ⭐⭐⭐ |

---

## 🔑 Columnas Clave por Tabla

### ⭐ i_accrual (DEVENGO_PRESTACION)

```sql
-- TABLA CENTRAL DEL SISTEMA
FROM atemplate.i_accrual AS iacc

-- Columnas esenciales:
iacc.datetrx                -- 📅 Fecha Transaccion
iacc.ex_episodevalue        -- 🏥 Codigo Episodio
iacc.bpartnervalue          -- 👤 Codigo Paciente
iacc.bpartnername           -- 👤 Nombre Paciente (desnorm)
iacc.productvalue           -- 💊 Codigo Prestacion
iacc.description            -- 💊 Descripcion Prestacion
iacc.datefrom               -- 📅 Fecha Prestacion Real
iacc.qtyordered             -- 🔢 Cantidad
iacc.priceactual            -- 💰 Monto Bruto
iacc.taxamt                 -- 💰 Monto IVA
iacc.linenetamt             -- 💰 Monto Neto (TOTAL)
iacc.ex_locale_name         -- 🏢 Centro Medico
iacc.payorvalue             -- 🏥 Codigo Aseguradora
iacc.payor_name             -- 🏥 Nombre Aseguradora (desnorm)
iacc.orgvalue               -- 🏛️ Codigo Organizacion
iacc.ex_mainaccount         -- 📊 Cuenta Contable
iacc.ex_processtype         -- ⚙️ NEW/CANCEL
```

**Joins típicos:**
```sql
-- Paciente
LEFT JOIN c_bpartner AS bp_paciente 
    ON bp_paciente.value = iacc.bpartnervalue

-- Aseguradora
LEFT JOIN c_bpartner AS bp_aseguradora 
    ON bp_aseguradora.value = iacc.payorvalue

-- Prestación
LEFT JOIN m_product AS prod 
    ON prod.value = iacc.productvalue
```

---

### 💳 c_invoice (FACTURA)

```sql
FROM atemplate.c_invoice AS inv

-- Columnas esenciales:
inv.documentno              -- 📄 Numero Factura
inv.dateinvoiced            -- 📅 Fecha Venta/Emision
inv.c_bpartner_id           -- 🔗 FK: ID Paciente
inv.c_doctype_id            -- 🔗 FK: ID Tipo Documento
inv.c_pos_id                -- 🔗 FK: ID Punto Venta
inv.ex_episode_value        -- 🏥 Codigo Episodio
inv.grandtotal              -- 💰 Monto Total
inv.docstatus               -- ⚙️ CO/DR/VO
inv.ispaid                  -- ✅ Pagada? Y/N
```

**Joins típicos:**
```sql
-- Líneas de factura
INNER JOIN c_invoiceline AS il 
    ON il.c_invoice_id = inv.c_invoice_id

-- Paciente
INNER JOIN c_bpartner AS bp 
    ON bp.c_bpartner_id = inv.c_bpartner_id

-- Tipo documento
INNER JOIN c_doctype AS dt 
    ON dt.c_doctype_id = inv.c_doctype_id

-- Punto de venta → Centro médico
INNER JOIN c_pos AS cpos 
    ON cpos.c_pos_id = inv.c_pos_id
INNER JOIN ex_locale AS loc 
    ON loc.ex_locale_id = cpos.ex_locale_id
```

---

### 📋 c_invoiceline (LINEA_FACTURA)

```sql
FROM atemplate.c_invoiceline AS il

-- Columnas esenciales:
il.c_invoice_id             -- 🔗 FK: ID Factura
il.m_product_id             -- 🔗 FK: ID Prestacion
il.qtyinvoiced              -- 🔢 Cantidad
il.priceactual              -- 💰 Precio Unitario
il.linenetamt               -- 💰 Monto Total Linea
il.ex_insuranceamount       -- 🏥 Monto Seguro
il.ex_copayment             -- 💳 Monto Copago
il.ex_surplusamount         -- 💸 Monto Excedente

-- VALIDACIÓN CRÍTICA:
-- linenetamt = ex_insuranceamount + ex_copayment + ex_surplusamount
```

---

### 🧾 ex_postransaction (TRANSACCION_CAJA)

```sql
FROM atemplate.ex_postransaction AS trx

-- Columnas esenciales:
trx.value                   -- 🧾 Numero Transaccion POS
trx.created                 -- 📅 Fecha Hora Transaccion
trx.created::date           -- 📅 Fecha Proceso (solo fecha)
trx.ex_possession_id        -- 🔗 FK: ID Caja
trx.c_bpartner_id           -- 🔗 FK: ID Paciente
trx.docstatus               -- ⚙️ CO/DR

-- Extraer hora:
to_char(trx.created, 'HH24:MI:SS') AS hora_proceso
```

**Joins típicos:**
```sql
-- Líneas de transacción
INNER JOIN ex_postransactionline AS trxline 
    ON trxline.ex_postransaction_id = trx.ex_postransaction_id

-- Caja
INNER JOIN ex_possession AS pos 
    ON pos.ex_possession_id = trx.ex_possession_id

-- Cajero (desde caja)
INNER JOIN ad_user AS usr 
    ON usr.ad_user_id = pos.createdby

-- Centro médico (cadena)
INNER JOIN c_pos AS cpos 
    ON cpos.c_pos_id = pos.c_pos_id
INNER JOIN ex_locale AS loc 
    ON loc.ex_locale_id = cpos.ex_locale_id
```

---

### 👤 c_bpartner (PACIENTE / ASEGURADORA)

```sql
FROM atemplate.c_bpartner AS bp

-- Columnas esenciales COMUNES:
bp.c_bpartner_id            -- 🔑 ID Unico
bp.value                    -- 📋 Codigo
bp.taxid                    -- 🆔 RUT
bp.name                     -- 👤 Nombre
bp.phone                    -- 📞 Telefono
bp.email                    -- 📧 Email

-- Solo PACIENTE:
bp.iscustomer               -- ✅ 'Y' para pacientes
bp.birthday                 -- 📅 Fecha Nacimiento
bp.gender                   -- ⚧ M/F
bp.bloodgroup               -- 🩸 Grupo Sanguineo
bp.ex_payor_id              -- 🔗 FK: ID Aseguradora

-- Distinguir:
WHERE bp.iscustomer = 'Y'   -- PACIENTE
-- o buscar en payorvalue    -- ASEGURADORA
```

---

## 🔗 Patrones de Join

### 📌 Patrón 1: Join por VALUE (código alfanumérico)
```sql
-- Común en i_accrual
i_accrual.bpartnervalue = c_bpartner.value
i_accrual.payorvalue = c_bpartner.value
i_accrual.productvalue = m_product.value
```
**⚠️ NO es join por ID, sino por código**

### 📌 Patrón 2: Join por ID (FK tradicional)
```sql
-- Común en facturas
c_invoiceline.c_invoice_id = c_invoice.c_invoice_id
c_invoice.c_bpartner_id = c_bpartner.c_bpartner_id
c_invoiceline.m_product_id = m_product.m_product_id
```

### 📌 Patrón 3: Join en Cadena (Navegación)
```sql
-- De Transacción a Centro Médico:
ex_postransaction
    → ex_possession (caja)
    → c_pos (punto de venta)
    → ex_locale (centro médico)
```

### 📌 Patrón 4: COALESCE para Alternativas
```sql
-- Medio de pago de dos fuentes posibles:
COALESCE(ex_tendertype.name, c_doctype.printname) AS medio_pago
```

---

## 🎯 Queries de Ejemplo Rápido

### Ejemplo 1: Devengos de un Paciente
```sql
SELECT 
    iacc.ex_episodevalue AS codigo_episodio,
    iacc.description AS prestacion,
    iacc.datefrom AS fecha,
    iacc.linenetamt AS monto
FROM atemplate.i_accrual iacc
LEFT JOIN atemplate.c_bpartner bp ON bp.value = iacc.bpartnervalue
WHERE bp.taxid = '12345678-9'
ORDER BY iacc.datefrom DESC;
```

### Ejemplo 2: Facturas del Día
```sql
SELECT 
    inv.documentno AS factura,
    bp.name AS paciente,
    inv.grandtotal AS total
FROM atemplate.c_invoice inv
INNER JOIN atemplate.c_bpartner bp ON bp.c_bpartner_id = inv.c_bpartner_id
WHERE inv.dateinvoiced = CURRENT_DATE
    AND inv.docstatus = 'CO';
```

### Ejemplo 3: Cierre de Caja
```sql
SELECT 
    pos.name AS caja,
    usr.name AS cajero,
    COUNT(*) AS transacciones,
    SUM(trxline.linenetamt) AS total
FROM atemplate.ex_postransaction trx
INNER JOIN atemplate.ex_postransactionline trxline 
    ON trxline.ex_postransaction_id = trx.ex_postransaction_id
INNER JOIN atemplate.ex_possession pos 
    ON pos.ex_possession_id = trx.ex_possession_id
INNER JOIN atemplate.ad_user usr 
    ON usr.ad_user_id = pos.createdby
WHERE DATE(trx.created) = CURRENT_DATE
    AND trx.docstatus = 'CO'
GROUP BY pos.name, usr.name;
```

---

## 📊 Estados y Valores Comunes

### docstatus (Estado de Documento)
```
'DR' = Draft        (Borrador)
'CO' = Completed    (Completado) ✅
'VO' = Voided       (Anulado) ❌
'CL' = Closed       (Cerrado)
```

### ex_processtype (Tipo de Proceso en Devengo)
```
'NEW'     = Cargo nuevo ✅
'CANCEL'  = Anulación ❌
```

### printname (Tipos de Documento)
```
'Boleta'
'Boleta Manual'
'Exempt Ticket'         (Boleta Exenta)
'Invoice'               (Factura)
'Factura Exenta'
'Credit Memo'           (Nota de Crédito)
'Nota de Credito (test)'
```

---

## 💰 Fórmulas Financieras Críticas

### En c_invoiceline
```sql
-- SIEMPRE debe cumplirse:
linenetamt = ex_insuranceamount + ex_copayment + ex_surplusamount

-- Validación:
SELECT 
    c_invoiceline_id,
    linenetamt AS total,
    (ex_insuranceamount + ex_copayment + ex_surplusamount) AS suma,
    (linenetamt - (ex_insuranceamount + ex_copayment + ex_surplusamount)) AS diferencia
FROM c_invoiceline
WHERE ABS(linenetamt - (ex_insuranceamount + ex_copayment + ex_surplusamount)) > 0.01;
```

### En i_accrual
```sql
-- Monto Neto = Monto Bruto + IVA
linenetamt = priceactual + taxamt
```

---

## 🏷️ Convenciones de Nomenclatura

### Prefijos de Campos
- `ex_*` = Extensiones GESNOVA (customizaciones)
- `c_*` = Core/Cliente (entidades principales)
- `m_*` = Maestros (catálogos)
- `ad_*` = Application Dictionary (admin)
- `i_*` = Information (vistas de negocio)

### Sufijos Comunes
- `*_id` = Clave primaria o foránea
- `*value` = Código alfanumérico
- `*name` = Nombre descriptivo
- `*amt` = Monto/Amount
- `*qty` = Cantidad/Quantity

---

## 🔍 Campos Desnormalizados en i_accrual

⚠️ **IMPORTANTE:** Estos campos están duplicados para performance:

```sql
i_accrual.bpartnername      ← c_bpartner.name
i_accrual.payor_name        ← c_bpartner.name (aseguradora)
i_accrual.ex_locale_name    ← ex_locale.name
i_accrual.description       ← m_product.name
```

**Ventaja:** Queries más rápidas  
**Desventaja:** Deben mantenerse sincronizados

---

## 📅 Filtros de Fecha Comunes

```sql
-- Rango de fechas (patrón recomendado)
WHERE datetrx >= TIMESTAMP '2025-10-01 00:00:00'
  AND datetrx <  TIMESTAMP '2025-11-01 00:00:00'

-- Solo fecha (sin timestamp)
WHERE dateinvoiced >= DATE '2025-10-01'
  AND dateinvoiced <  DATE '2025-11-01'

-- Día actual
WHERE DATE(created) = CURRENT_DATE

-- Mes actual
WHERE DATE_TRUNC('month', datetrx) = DATE_TRUNC('month', CURRENT_DATE)
```

---

## 🧩 Diagrama de Relaciones Simplificado

```
ORGANIZACION (ad_org)
    ↓
    ├─→ PACIENTE (c_bpartner where iscustomer='Y')
    ├─→ CENTRO_MEDICO (ex_locale)
    └─→ PUNTO_VENTA (c_pos) → CAJA (ex_possession)

PACIENTE
    ├─→ EPISODIO (ex_episodevalue en i_accrual)
    │       ↓
    │   DEVENGO (i_accrual) ← PRESTACION (m_product)
    │       ↓                 ↑
    │   FACTURA (c_invoice)   │
    │       ↓                 │
    │   LINEA_FACTURA ────────┘
    │       (c_invoiceline)
    └─→ TRANSACCION_CAJA (ex_postransaction)
            ↓
        LINEA_TRANSACCION (ex_postransactionline)
```

---

## 🎓 Tips Pro

### 1. Siempre validar estado
```sql
WHERE docstatus = 'CO'  -- Solo documentos completados
```

### 2. Usar COALESCE para nulls
```sql
COALESCE(tend.name, dt.printname, 'Sin especificar') AS medio_pago
```

### 3. Formatear montos
```sql
TO_CHAR(linenetamt, 'FM999,999,999') AS monto_formateado
```

### 4. Agrupar por periodo
```sql
DATE_TRUNC('month', datetrx)::DATE AS periodo_mes
```

### 5. Verificar tipos de documento
```sql
WHERE dt.printname IN ('Boleta', 'Invoice', 'Factura Exenta')
```

---

## 🚨 Errores Comunes

### ❌ ERROR 1: Join incorrecto en i_accrual
```sql
-- MAL:
JOIN c_bpartner ON c_bpartner.c_bpartner_id = i_accrual.????

-- BIEN:
JOIN c_bpartner ON c_bpartner.value = i_accrual.bpartnervalue
```

### ❌ ERROR 2: Olvidar filtrar estado
```sql
-- MAL: incluye borradores y anulados
SELECT * FROM c_invoice

-- BIEN:
SELECT * FROM c_invoice WHERE docstatus = 'CO'
```

### ❌ ERROR 3: No validar suma de montos
```sql
-- MAL: asumir que la suma está bien
SELECT linenetamt FROM c_invoiceline

-- BIEN: validar
SELECT 
    linenetamt,
    (ex_insuranceamount + ex_copayment + ex_surplusamount) AS verificacion
FROM c_invoiceline
WHERE ABS(linenetamt - (ex_insuranceamount + ex_copayment + ex_surplusamount)) > 0.01
```

---

## 📚 Referencias Rápidas

| Necesito... | Ver documento... |
|------------|------------------|
| Queries reales completas | `documentacion_hibrida_completa.md` |
| Modelo visual | `modelo_conceptual_hibrido_gesnova.mermaid` |
| Alias SQL | `alias_semanticos_gesnova.sql` |
| Guía de uso | `guia_rapida_alias_semanticos.md` |

---

## 🎯 Casos de Uso Ultra-Rápidos

### "Necesito devengos de un episodio"
```sql
SELECT * FROM atemplate.i_accrual 
WHERE ex_episodevalue = 'EP-2025-001';
```

### "Necesito facturas de un paciente"
```sql
SELECT inv.* FROM atemplate.c_invoice inv
JOIN atemplate.c_bpartner bp ON bp.c_bpartner_id = inv.c_bpartner_id
WHERE bp.taxid = '12345678-9';
```

### "Necesito ventas del día"
```sql
SELECT documentno, grandtotal FROM atemplate.c_invoice
WHERE dateinvoiced = CURRENT_DATE AND docstatus = 'CO';
```

### "Necesito cierre de caja"
```sql
SELECT SUM(linenetamt) FROM atemplate.ex_postransactionline trxline
JOIN atemplate.ex_postransaction trx ON trx.ex_postransaction_id = trxline.ex_postransaction_id
WHERE DATE(trx.created) = CURRENT_DATE AND trx.docstatus = 'CO';
```

---

**FIN DEL CHEAT SHEET**

*Imprime esto y pégalo en tu escritorio* 📌
