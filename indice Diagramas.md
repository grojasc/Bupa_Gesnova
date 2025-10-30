# 📊 Índice de Diagramas GESNOVA

## Guía de Visualización del Modelo de Datos

---

## 🎯 Diagramas Disponibles

### 1. **Modelo Conceptual Híbrido Completo** ⭐
**Archivo:** `modelo_conceptual_hibrido_gesnova_v2.mermaid`

**Descripción:**  
Diagrama de entidad-relación (ERD) completo que muestra TODAS las entidades del sistema con doble nomenclatura:
- Nombre semántico (concepto de negocio)
- Nombre físico (tabla en base de datos)

**Incluye:**
- ✅ 20 entidades principales
- ✅ Todas las relaciones entre tablas
- ✅ Atributos clave de cada entidad con nombres físicos
- ✅ Tipos de datos (PK, FK)
- ✅ Ejemplos de valores

**Cuándo usar:**
- Para entender el modelo completo
- Para diseñar nuevas funcionalidades
- Para documentación técnica
- Para capacitación de desarrolladores

**Entidades incluidas:**
```
• ORGANIZACION (ad_org)
• PACIENTE (c_bpartner)
• ASEGURADORA (c_bpartner)
• EPISODIO_CLINICO (derivado)
• DEVENGO_PRESTACION (i_accrual) ⭐ CENTRAL
• PRESTACION_MEDICA (m_product)
• CATEGORIA_PRESTACION (m_product_category)
• CENTRO_MEDICO (ex_locale)
• FACTURA (c_invoice)
• LINEA_FACTURA (c_invoiceline)
• TIPO_DOCUMENTO (c_doctype)
• TRANSACCION_CAJA (ex_postransaction)
• LINEA_TRANSACCION (ex_postransactionline)
• CAJA (ex_possession)
• PUNTO_VENTA (c_pos)
• CAJERO (ad_user)
• TIPO_PAGO (ex_tendertype)
• PAGO (c_payment)
```

---

### 2. **Modelo Simplificado** 🎯
**Archivo:** `modelo_simplificado_gesnova.mermaid`

**Descripción:**  
Versión reducida del modelo que muestra solo las entidades y relaciones más importantes para un entendimiento rápido.

**Incluye:**
- ✅ 10 entidades principales
- ✅ Relaciones clave
- ✅ Atributos esenciales
- ✅ Nomenclatura híbrida

**Cuándo usar:**
- Para presentaciones ejecutivas
- Para entendimiento rápido
- Para nuevos miembros del equipo
- Para documentación de alto nivel

**Entidades incluidas:**
```
• PACIENTE (c_bpartner)
• ASEGURADORA (c_bpartner)
• DEVENGO_PRESTACION (i_accrual) ⭐
• PRESTACION_MEDICA (m_product)
• CENTRO_MEDICO (ex_locale)
• FACTURA (c_invoice)
• LINEA_FACTURA (c_invoiceline)
• TRANSACCION_CAJA (ex_postransaction)
• CAJA (ex_possession)
• ORGANIZACION (ad_org)
```

**Relaciones destacadas:**
- PACIENTE → DEVENGO_PRESTACION (recibe prestaciones)
- DEVENGO_PRESTACION → PRESTACION_MEDICA (servicio realizado)
- FACTURA → LINEA_FACTURA (contiene líneas)
- TRANSACCION_CAJA → CAJA (registrada en)

---

### 3. **Flujo de Procesos de Negocio** 🔄
**Archivo:** `flujo_procesos_gesnova.mermaid`

**Descripción:**  
Diagrama de flujo que muestra el proceso completo desde que el paciente llega hasta que se completa el pago, identificando en qué tabla se registra cada paso.

**Incluye:**
- ✅ 4 flujos principales
- ✅ Puntos de decisión
- ✅ Validaciones críticas
- ✅ Mapeo a tablas físicas

**Cuándo usar:**
- Para entender el flujo operativo
- Para capacitación de usuarios
- Para identificar cuellos de botella
- Para diseñar mejoras de proceso

**Flujos incluidos:**

#### **FLUJO 1: Registro y Atención**
```
1. Paciente llega
2. Registro en PACIENTE (c_bpartner)
3. ¿Tiene aseguradora? → Vincular ex_payor_id
4. Apertura de EPISODIO (ex_episodevalue)
```

#### **FLUJO 2: Prestaciones**
```
1. Prestación médica realizada
2. Registro en DEVENGO_PRESTACION (i_accrual)
   • codigo_episodio
   • codigo_paciente
   • codigo_prestacion
   • monto_neto
   • centro_medico
   • codigo_aseguradora
3. ¿Más prestaciones? → Repetir o cerrar episodio
```

#### **FLUJO 3: Facturación**
```
1. Generación de FACTURA (c_invoice)
2. Crear LINEAS_FACTURA (c_invoiceline)
3. Calcular desglose:
   • monto_seguro
   • monto_copago
   • monto_excedente
4. Validación: suma = total
5. Factura completada (docstatus = 'CO')
```

#### **FLUJO 4: Cobro**
```
1. Paciente va a CAJA (ex_possession)
2. CAJERO atiende (ad_user)
3. Crear TRANSACCION_CAJA (ex_postransaction)
4. Registrar medio de pago (ex_postransactionline)
5. Aplicar pago a FACTURA
6. Actualizar estado: ispaid = 'Y'
```

**Códigos de color:**
- 🔵 Azul: Registro en tabla
- 🟠 Naranja: Proceso/Acción
- 🔴 Rosa: Decisión/Validación
- 🔴 Rojo: Error

---

### 4. **Diagrama de Queries Principales** 🔍
**Archivo:** `diagrama_queries_principales.mermaid`

**Descripción:**  
Muestra visualmente las 4 queries reales de producción, identificando qué tablas se unen y cómo.

**Incluye:**
- ✅ 4 queries documentadas
- ✅ Joins entre tablas
- ✅ Campos clave usados
- ✅ Resultados generados

**Cuándo usar:**
- Para escribir queries similares
- Para entender patrones de join
- Para optimización de consultas
- Para debugging

**Queries incluidas:**

#### **QUERY 1: Devengado** (Devengado.sql)
```
Tablas principales:
• i_accrual (DEVENGO_PRESTACION) ⭐ principal
• c_bpartner (PACIENTE) → join por bpartnervalue
• c_bpartner (ASEGURADORA) → join por payorvalue

Resultado:
• Codigo Episodio
• RUT Paciente
• Prestacion
• Monto Neto
• Centro Medico
• Aseguradora
```

#### **QUERY 2: Venta Online POS** (Venta_Online_2.sql)
```
Tablas principales:
• ex_postransaction (TRANSACCION) ⭐ principal
• ex_postransactionline (LINEA)
• ex_possession (CAJA)
• c_pos (PUNTO_VENTA)
• ex_locale (CENTRO_MEDICO)
• ad_user (CAJERO) → join desde possession.createdby
• c_bpartner (PACIENTE)

Resultado:
• Numero Transaccion
• Fecha Hora
• RUT Paciente
• Centro Medico
• Usuario Cajero
• Monto Neto
```

#### **QUERY 3: Recaudación** (Recaudacion.sql)
```
Tablas principales:
• ex_postransaction ⭐ principal
• ex_postransactionline
• c_payment (PAGO) → LEFT JOIN
• ex_tendertype (TIPO_PAGO)
• c_invoice (FACTURA) → LEFT JOIN
• c_doctype (TIPO_DOC)

Medio de Pago (COALESCE):
• ex_tendertype.name (si hay pago directo)
• c_doctype.printname (si referencia factura)

Resultado:
• Numero Transaccion
• Fecha Hora
• RUT Paciente
• Centro Medico
• Medio de Pago (COALESCE)
• Monto Aplicado
```

#### **QUERY 4: Venta IntegraMédica** (Venta_Integramedica.sql)
```
Tablas principales:
• c_invoiceline (LINEA_FACTURA) ⭐ principal
• c_invoice (FACTURA)
• c_bpartner (PACIENTE)
• m_product (PRESTACION)
• c_doctype (TIPO_DOC)
• c_pos (POS) → ex_locale (CENTRO)

Desglose Financiero:
• ex_insuranceamount (monto_seguro)
• ex_copayment (monto_copago)
• ex_surplusamount (monto_excedente)

Validación Crítica:
linenetamt = ex_insuranceamount + ex_copayment + ex_surplusamount

Resultado:
• Numero Factura
• Tipo Documento
• Fecha Venta
• RUT Paciente
• Prestacion
• Desgloses financieros
• Centro Medico
```

**Códigos de color:**
- 🔵 Azul oscuro: Tabla principal de la query
- 🔵 Azul claro: Tablas join
- 🟢 Verde: Resultado final
- 🟠 Naranja: Validaciones especiales

---

## 🛠️ Cómo Visualizar los Diagramas

### Opción 1: Visual Studio Code
```bash
# Instalar extensión
# Buscar: "Markdown Preview Mermaid Support"
# Luego abrir el archivo .mermaid y presionar Ctrl+Shift+V
```

### Opción 2: Mermaid Live Editor (Online)
```
1. Ir a: https://mermaid.live/
2. Copiar el contenido del archivo .mermaid
3. Pegar en el editor
4. Ver el diagrama renderizado
5. Exportar como PNG/SVG si es necesario
```

### Opción 3: GitHub
```
# GitHub renderiza automáticamente archivos .mermaid
# Solo súbelos al repositorio y ábrelos en GitHub
```

### Opción 4: Markdown con Mermaid
```markdown
# En cualquier archivo .md puedes incluir:

```mermaid
[contenido del archivo .mermaid]
```
```

---

## 📋 Tabla Comparativa de Diagramas

| Diagrama | Complejidad | Detalle | Propósito | Audiencia |
|----------|-------------|---------|-----------|-----------|
| **Modelo Completo** | Alta | Máximo | Documentación técnica | Desarrolladores |
| **Modelo Simplificado** | Baja | Medio | Entendimiento rápido | Todos |
| **Flujo de Procesos** | Media | Alto | Procesos de negocio | Usuarios/Analistas |
| **Diagrama de Queries** | Alta | Máximo | Desarrollo de queries | Desarrolladores/DBAs |

---

## 🎯 Guía de Uso por Rol

### Para Desarrolladores 👨‍💻
```
1. Empieza con: Modelo Simplificado
2. Luego estudia: Modelo Completo
3. Usa para queries: Diagrama de Queries
4. Referencia: documentacion_hibrida_completa.md
```

### Para Analistas de Negocio 📊
```
1. Empieza con: Flujo de Procesos
2. Luego revisa: Modelo Simplificado
3. Para dudas: cheat_sheet_gesnova.md
```

### Para Gerentes/Ejecutivos 👔
```
1. Solo revisa: Modelo Simplificado
2. Para procesos: Flujo de Procesos
```

### Para DBAs 🗄️
```
1. Estudia: Modelo Completo
2. Optimiza con: Diagrama de Queries
3. Implementa: alias_semanticos_gesnova.sql
```

---

## 🔄 Mantenimiento de Diagramas

### Cuándo Actualizar

Actualiza los diagramas cuando:
- ✅ Se agreguen nuevas tablas
- ✅ Se modifiquen relaciones
- ✅ Se creen nuevas queries importantes
- ✅ Cambien los procesos de negocio

### Cómo Actualizar

1. **Modelo Completo:**
   - Agregar nueva entidad con sus atributos
   - Definir relaciones con entidades existentes
   - Incluir nombre físico de tabla

2. **Modelo Simplificado:**
   - Solo agregar si es entidad muy importante
   - Mantener máximo 12 entidades

3. **Flujo de Procesos:**
   - Actualizar cuando cambien procesos operativos
   - Mantener 4-5 flujos principales

4. **Diagrama de Queries:**
   - Agregar nuevas queries que se usen frecuentemente
   - Mantener máximo 6 queries

---

## 📚 Documentos Relacionados

| Documento | Descripción |
|-----------|-------------|
| `documentacion_hibrida_completa.md` | Documentación detallada con análisis de queries |
| `cheat_sheet_gesnova.md` | Referencia rápida para developers |
| `alias_semanticos_gesnova.sql` | Vistas SQL con nombres semánticos |
| `guia_rapida_alias_semanticos.md` | Cómo usar las vistas SQL |

---

## 🎨 Leyenda de Símbolos

### En Diagramas ERD:
```
||--o{   Uno a Muchos (1:N)
}o--||   Muchos a Uno (N:1)
}o--o{   Muchos a Muchos (N:M)
||--|{   Uno a Muchos obligatorio
}o--o|   Muchos a Uno opcional
```

### En Atributos:
```
PK      = Primary Key (Llave primaria)
FK      = Foreign Key (Llave foránea)
string  = Texto
int     = Entero
decimal = Número con decimales
date    = Fecha
timestamp = Fecha y hora
bool    = Booleano (Y/N, true/false)
```

### En Flujos:
```
[]      = Proceso/Acción
{}      = Decisión
()      = Inicio/Fin
```

---

## 💡 Tips para Mejor Comprensión

### 1. **Lee en orden**
```
Modelo Simplificado → Flujo Procesos → Modelo Completo → Queries
```

### 2. **Identifica patrones**
```
• c_bpartner sirve para PACIENTE y ASEGURADORA
• Joins por VALUE en i_accrual
• COALESCE para alternativas
• Validaciones financieras en c_invoiceline
```

### 3. **Usa códigos de color**
```
• Azul = Tablas
• Verde = Resultados
• Naranja = Validaciones
• Rojo = Errores
```

### 4. **Enfócate en la tabla central**
```
i_accrual (DEVENGO_PRESTACION) es la columna vertebral
Desde ahí se conecta todo
```

---

## 🚀 Próximos Pasos

1. ✅ Visualiza cada diagrama
2. ✅ Lee la documentación completa
3. ✅ Practica con las queries de ejemplo
4. ✅ Implementa las vistas SQL
5. ✅ Usa el cheat sheet como referencia

---

**¡Disfruta explorando el modelo GESNOVA!** 🎉

*Última actualización: Octubre 2025*
