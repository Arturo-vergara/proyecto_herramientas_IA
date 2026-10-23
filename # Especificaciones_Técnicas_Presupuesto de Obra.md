# Especificaciones Técnicas para Automatización: Normalización Redes Contra Incendio

**Proyecto:** Normalización Redes Contra Incendio del Recinto
**Empresa:** CONSTRUCTORA IA_GEN S.A.
**Fecha de Creación:** 2026-10-20
**Versión:** 1.1

## 1. Introducción

Este documento detalla las especificaciones técnicas para automatizar la tarea de completar el presupuesto del proyecto "Normalización Redes Contra Incendio del Recinto", utilizando información de precios de materiales y generales. El proceso se centrará en poblar el archivo `ITEMIZADO_PRESUPUESTO_A_COMPLETAR.xlsx` con los costos unitarios y totales correspondientes, basándose en el archivo `PRECIO_MATERIALES.xlsx`. La automatización se realizará mediante agentes de IA, principalmente scripts de Python.

## 2. Objetivos de la Automatización

*   Completar los campos de `Costo Unitario` y `Costo Total por Ítem` en el archivo `ITEMIZADO_PRESUPUESTO_A_COMPLETAR.xlsx`.
*   Asociar cada ítem del presupuesto con su precio unitario correspondiente del archivo `PRECIO_MATERIALES.xlsx`.
*   Calcular el costo total por ítem: `Costo Total = Cantidad × Costo Unitario`.
*   Calcular los agregados del presupuesto según las reglas establecidas: Gastos Generales, Utilidades e IVA.
*   Identificar ítems del presupuesto que no tengan un precio correspondiente en la base de datos de precios.
*   Generar un reporte final con los resultados, costos totales y observaciones.

## 3. Fuentes de Datos y Preprocesamiento

### 3.1. Archivo: ITEMIZADO_PRESUPUESTO_A_COMPLETAR.xlsx

*   **Descripción:** Contiene el detalle de los ítems necesarios para la normalización de las redes contra incendio.
*   **Columnas Clave:**
    *   `Nº / Ítem`: Identificador y descripción del elemento.
    *   `Uni`: Unidad de medida (UN, MTS, M2, KG, M3, GL, etc.).
    *   `Cantidad`: Cantidad requerida.
    *   `P. Unitario`: Precio unitario a completar.
    *   `P. Total`: Costo total por ítem a calcular.
    *   `NO ENCONTRADO`: Columna para marcar ítems sin precio asociado.
*   **Preprocesamiento:**
    *   Cargar archivo Excel, preservando la estructura jerárquica por capítulos.
    *   Convertir cantidades a formato numérico.
    *   Normalizar descripciones para búsqueda en la base de precios.

### 3.2. Archivo: PRECIO_MATERIALES.xlsx

*   **Descripción:** Contiene precios unitarios de materiales, equipos y actividades generales.
*   **Columnas Clave:**
    *   `Ítem`: Nombre del material o actividad.
    *   `Uni`: Unidad de medida.
    *   `P. Unitario`: Valor unitario en moneda nacional.
*   **Preprocesamiento:**
    *   Unificar en una sola tabla las secciones de materiales y precios generales.
    *   Convertir precios a formato numérico consistente.
    *   Normalizar nombres para coincidencia con el presupuesto.

## 4. Reglas de Cálculo del Presupuesto

Se aplicarán obligatoriamente las siguientes reglas de agregación:

| Concepto | Fórmula | Porcentaje |
|---|---|---|
| **Costo Directo Total (CDT)** | Suma de todos los `P. Total` de ítems | — |
| **Gastos Generales** | CDT × 15% | 15% del Costo Directo Total |
| **Utilidades** | CDT × 15% | 15% del Costo Directo Total |
| **Subtotal / Costo Total antes de Impuesto** | CDT + Gastos Generales + Utilidades | — |
| **IVA** | Subtotal × 19% | 19% del Subtotal |
| **VALOR TOTAL DEL PROYECTO** | Subtotal + IVA | — |

> **Nota:** Los Gastos Generales y Utilidades se calculan cada uno sobre el **Costo Directo Total**, no se suman entre sí antes de aplicar.

## 5. Especificaciones para Agentes de IA

### 5.1. Agente de Carga y Limpieza de Datos

*   **Función:** Cargar ambos archivos, unificar tablas de precios y normalizar formatos.
*   **Entrada:** Rutas de los archivos Excel.
*   **Salida:** Dos estructuras de datos limpias: `presupuesto` y `precios`.
*   **Validaciones:**
    *   Verificar que cantidades sean mayores o iguales a cero.
    *   Detectar valores no numéricos en columnas de precio.
    *   Registrar discrepancias de formato.

### 5.2. Agente de Búsqueda y Llenado de Precios

*   **Función:** Asignar precios unitarios y calcular totales por ítem.
*   **Lógica:**
    1. Por cada ítem del presupuesto, buscar coincidencia en la tabla de precios comparando **descripción + unidad**.
    2. Si hay coincidencia → copiar `P. Unitario` y calcular:
       > `P. Total = Cantidad × P. Unitario`
    3. Si no hay coincidencia → marcar columna `NO ENCONTRADO = "SÍ"` y dejar montos vacíos.
    4. Usar coincidencia aproximada si las descripciones no son idénticas pero coinciden en términos clave.
*   **Salida:** Presupuesto con montos poblados y columna de estado.

### 5.3. Agente de Cálculo de Totales y Agregados

*   **Función:** Ejecutar las reglas de la Sección 4 sobre el presupuesto completado.
*   **Secuencia:**
    1. Sumar todos los `P. Total` → **Costo Directo Total**.
    2. Calcular:
       - Gastos Generales = CDT × 0.15
       - Utilidades = CDT × 0.15
    3. Sumar: CDT + Gastos Generales + Utilidades → **Subtotal**.
    4. Calcular IVA = Subtotal × 0.19.
    5. Sumar Subtotal + IVA → **Valor Total del Proyecto**.
*   **Salida:** Montos finales listos para insertar al pie del presupuesto.

### 5.4. Agente de Generación de Reporte

*   **Función:** Exportar resultados a archivo `.md` y a Excel final.
*   **Contenido del Reporte:**
    *   Resumen de ítems procesados vs. sin precio.
    *   Desglose detallado:
      - Costo Directo Total
      - Gastos Generales (15%)
      - Utilidades (15%)
      - Subtotal
      - IVA (19%)
      - **VALOR TOTAL**
    *   Listado de ítems sin precio asignado para revisión manual.
    *   Observaciones sobre formato o datos inconsistentes.

## 6. Requisitos Técnicos

*   **Lenguaje:** Python 3.x
*   **Librerías sugeridas:**
    - `pandas` → manipulación de datos
    - `openpyxl` → lectura/escritura de Excel
    - `thefuzz` → coincidencia aproximada de descripciones
*   **Salida final:**
    - Archivo Excel completado: `PRESUPUESTO_FINAL_COMPLETADO.xlsx`
    - Archivo de reporte: `REPORTE_AUTOMATIZACION_PRESUPUESTO.md`

## 7. Control de Versiones

| Versión | Fecha | Cambio Realizado |
|---|---|---|
| 1.0 | 2026-10-20 | Versión inicial |
| 1.1 | 2026-10-20 | Incorporadas reglas: Gastos Generales 15%, Utilidades 15%, IVA 19% |
