---
sidebar_position: 7
title: JUSTIFICACION ARQUITECNTONICA
---


---

## Visión General

El sistema sigue el proceso secuencial:
```
INICIO → Creación de inspección → Asignación de técnico → Técnico acepta
       → Ejecución → Registro de resultados → Cierre → FIN
```

Cada etapa genera un tipo distinto de dato. La arquitectura responde a esta realidad separando los **datos oficiales** (SQL) de los **datos operativos** (NoSQL), siguiendo las dos fases naturales del negocio: trabajo activo y certificación final.

---

## Datos en SQL: Estados y Responsabilidades Oficiales

SQL interviene en los momentos del flujo donde el sistema cambia responsabilidades o registra estados con valor legal.

### Etapas del flujo cubiertas por SQL

| Etapa | Razón |
|---|---|
| **Creación de inspección** | La entidad queda registrada oficialmente; existe obligación operativa |
| **Asignación de técnico** | Se designa un responsable único; no pueden coexistir dos asignaciones |
| **Técnico acepta** | La responsabilidad se transfiere mediante una transacción atómica |
| **Cierre de inspección** | Se genera el resultado final inmutable como evidencia auditable |

### Datos persistidos

- Inspección y sus metadatos
- Técnico asignado
- Estados del ciclo de vida
- Fecha de cierre
- Resultado final
- Historial oficial de cambios

### Fundamento

Estos eventos alteran el **estado legal del sistema**. Deben ser consistentes, únicos y verificables en cualquier auditoría futura.

---

## Datos en NoSQL: Trabajo Operativo en Tiempo Real

NoSQL interviene exclusivamente durante la etapa de ejecución, cuando el técnico trabaja activamente sobre la inspección.

### Etapa del flujo cubierta por NoSQL

**Ejecución de prueba / Registro de resultados**

### Datos generados

- Respuestas del checklist
- Observaciones libres
- Fotografías adjuntas
- Mediciones registradas
- Cambios y correcciones iterativas
- Autoguardados intermedios
- Estado de progreso

### Fundamento

Durante la ejecución el técnico puede corregir, repetir pasos, cancelar acciones y generar cientos de registros en minutos. Estos datos **no son definitivos hasta el cierre**. NoSQL permite escritura rápida y flexible sin interferir con la base principal ni bloquear otros procesos administrativos.

---

## Riesgos de Modelos Puros

### Si todo fuera relacional (solo SQL)

Llevar la ejecución a SQL introduciría contención operativa severa:

- Cada respuesta del checklist bloquearía la fila de inspección.
- Las operaciones de cierre se volverían lentas por el volumen de registros.
- La asignación de técnicos competiría con la escritura de resultados.

> El flujo operativo dañaría directamente el flujo administrativo.

### Si todo fuera documental (solo NoSQL)

Eliminar SQL como capa de control eliminaría las garantías del negocio:

- Dos técnicos podrían aceptar la misma inspección simultáneamente.
- Una inspección podría cerrarse sin responsable registrado.
- Los resultados podrían modificarse después del cierre.

> El flujo perdería toda validez auditable.

---

## Punto de Conexión entre Ambos Modelos

La integración ocurre en el momento del **cierre de inspección**, donde convergen ambas capas:
```
1. El sistema lee los datos de ejecución acumulados en NoSQL
2. Calcula el resultado final de la inspección
3. Persiste ese resultado como registro oficial en SQL
```
```
NoSQL  →  produce información durante la ejecución
SQL    →  certifica el resultado al momento del cierre
```

Este es el único punto de sincronización, y ocurre exactamente cuando los datos dejan de ser operativos para convertirse en evidencia.

---

## Consistencia y Trazabilidad

La arquitectura aplica una regla clara derivada del flujo:

| Fase | Estado de los datos | Motor |
|---|---|---|
| Durante la ejecución | Editable y mutable | **NoSQL** |
| Después del cierre | Inmutable y certificado | **SQL** |

Esta distinción garantiza que el técnico pueda trabajar sin restricciones mientras la empresa mantiene confianza total en el resultado final.

---

## El Modelo de Negocio como Origen del Diseño

El negocio no consiste en guardar formularios. Consiste en **certificar que una inspección ocurrió correctamente**.

El flujo tiene dos fases inherentes:
```
Fase de trabajo       →  flexible, iterativa, de alta frecuencia de escritura
Fase de certificación →  estricta, atómica, con garantía de integridad
```

La arquitectura híbrida SQL + NoSQL refleja exactamente esa dualidad, sin imponer restricciones donde se necesita flexibilidad ni relajar controles donde se exige rigor.

---

## Conclusión

El análisis del flujo de inspección revela dos realidades coexistentes:

- **Operación en tiempo real**: alta variabilidad, escritura frecuente, datos provisionales.
- **Validación final**: consistencia fuerte, unicidad, trazabilidad legal.

Separar SQL y NoSQL por responsabilidad, y no por conveniencia técnica, es lo que permite que el técnico trabaje con rapidez y la empresa confíe en el resultado.

> La arquitectura no es una elección de tecnología. Es una consecuencia directa del negocio.