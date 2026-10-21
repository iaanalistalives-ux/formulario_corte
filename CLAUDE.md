# CLAUDE.md — Corte: Eficiencia e Incentivos (formulario_corte)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta. **Corte es el proyecto piloto de la migración a SQL Server.**

## Qué hace
- `index.html` — **registro del día**. Tres tipos:
  - `MANUAL` (Manual de Corte): colaborador, operaciones con cantidad y minutos trabajados reales.
  - `CORTE_PAÑO`: uno o dos colaboradores con sus minutos y las operaciones del bloque (la eficiencia es del bloque).
  - `MODULO` (MÓD-1, MÓD-2, MÓD-3): operarios con minutos trabajados e improductivos (una vez por día), O/C con estilo, prendas y T. STD por pieza, y operaciones sueltas.
  - Cada guardado lleva un `token` para no duplicar ("Ya estaba guardado… No se duplicó"). Debajo, "Lo que va de la semana" por colaborador.
- `dashboard_corte.html` — **"CORTE — Eficiencia e Incentivos"**: semana y tipo; pestañas Real vs. Sistema, Semana a semana y Detalle por colaborador; KPIs de EF real, EF sistema, incentivo real, incentivo sistema y diferencia. Avisa si la semana no tiene cargado el extracto del sistema. Excel LIVES e Imprimir.
- `panel_corte.html` — **panel**: Registros de la semana (corregir y anular con motivo; nada se borra), Planilla semanal EF real vs. EF sistema (reemplaza a la pestaña PLANILLA_CORTE del Sheet), Personal y operaciones (catálogos) y, solo administración, **Sistema (BD_MINTRAB)**: se pega el texto copiado del Excel del SIGE (desde Cod.Linea hasta Incentivo Semanal, 7 días × minutos, eficiencia e incentivo), se revisa y se sube la semana (si existe, se reemplaza completa).

## Cálculo (lo hace la base)
- EF = minutos producidos ÷ minutos trabajados. Minutos producidos = cantidad × T. STD vigente al guardar (cada registro guarda su T. STD; cambiar la categoría de una persona sí recalcula todas las semanas).
- MODULO: EF del módulo por día = producción ÷ (minutos trabajados − improductivos), asignada a cada persona según sus minutos.
- Incentivo diario = % de la tabla de incentivos según la EF del día × básico de la categoría ÷ 30. Bajo 55 % no hay incentivo; desde 90 % se paga el máximo.
- EF Sistema solo cuenta los días que también tienen registro digitado. En el dashboard las sumas son ponderadas.

## Base de datos (Supabase lives-datos)
- Una sola puerta: `corte_api(p_accion, p)`.
  - Leer: `listas`, `catalogos`, `resumenSemana` (por `fecha` o `semana`), `semanasDisponibles`, `eficiencias`, `registros`, `mintrabSemanas`, `mintrabSemana`.
  - Registrar: `guardarManual`, `guardarCortePano`, `guardarModulo`, `corregir`, `anular`, `guardarPersona`, `guardarOperacion`, `subirMintrab`.
- Tablas: `corte_registros`, `corte_registro_personas`, `corte_registro_operaciones`, `corte_registro_oc`, `corte_personal`, `corte_operaciones_std`, `corte_incentivos`, `corte_bd_mintrab`, `corte_bitacora` (antes y después de cada cambio). Vistas de cálculo `corte_v_eficiencias` y `corte_v_ef_semana`.
- Los scripts SQL (C01, C02 v2, C04) no están en este repo. No usa Storage.
- Al 21/10/2026 todavía no hay registros guardados (solo catálogos).

## Quién la usa
Según las entradas por el portal de los últimos 30 días al 21/10/2026: 1 persona (Ingeniería en Corte), además de quien digita.

## Quién puede registrar
- Lo decide la base: `corte_puede_escribir_` (registrar, corregir, anular, catálogos) y `corte_puede_sistema_` (subir BD_MINTRAB), con la tabla `permisos`. `catalogos` devuelve `puedeEscribir` y `esAdmin` para mostrar u ocultar botones.
- A Sistemas: esas dos funciones son las piezas que reemplazan con su seguridad.

## Pendientes vistos al escribir esto (21/10/2026)
- Falta el paquete para Sistemas en el repo (script SQL Server, contrato de cada acción, diferencias con Postgres).
- `corregir` no lleva token de no-duplicado.
