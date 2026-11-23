---
theme: seriph
background: https://images.unsplash.com/photo-1517241034903-9a4c3ab12f00?q=80&w=1740&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
title: Herencia en EF Core — TPT · TPC · TPH
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
css: unocss
---

# Herencia en EF Core
## Diseño de Aplicaciones 1

<div class="pt-8 opacity-80">

TPT · TPC · TPH — cómo mapear herencia a tablas

</div>

---
layout: default
---

# El problema

<div class="text-lg mt-6">

El modelo **orientado a objetos** tiene herencia. El modelo **relacional** no.

Hay **tres formas** de mapear la misma jerarquía a tablas:

</div>

<div class="grid grid-cols-3 gap-4 mt-8 text-sm">

<div class="p-4 rounded bg-amber-500/10 border border-amber-500/30">

### TPT
*Table Per Type*

Una tabla por **clase**: la padre y cada hija.

</div>

<div class="p-4 rounded bg-emerald-500/10 border border-emerald-500/30">

### TPC
*Table Per Concrete type*

Una tabla por **clase concreta**, sin tabla padre.

</div>

<div class="p-4 rounded bg-cyan-500/10 border border-cyan-500/30">

### TPH
*Table Per Hierarchy*

**Una sola** tabla para toda la jerarquía.

</div>

</div>

<!--
No hay una "correcta": hay trade-offs.
-->

---
layout: default
---

# La jerarquía que queremos mapear

<div class="flex items-center justify-center h-4/5">

<img src="/herencia-ef.png" class="max-h-full max-w-full object-contain rounded bg-white p-2 shadow" />

</div>

<!--
Persona es la clase base. Funcionario, Docente y Estudiante heredan de ella.
Preguntar: ¿cuántas tablas necesitamos? Ahí arranca la discusión.
-->

---
layout: section
---

# TPT
## Table Per Type

---
layout: default
---

# TPT — Cómo quedarían las tablas

<div class="flex items-center justify-center h-3/5 mt-2">

<img src="/TPT.png" class="max-h-full max-w-full object-contain rounded bg-white p-2 shadow" />

</div>

<div class="mt-4 p-3 rounded bg-amber-500/10 border border-amber-500/30 text-xs">

<b>4 tablas.</b> El <code>id</code> de cada hija es PK y FK a <code>Persona</code>. Para traer un <code>Docente</code> completo hay que hacer JOIN entre <code>Persona</code> y <code>Docente</code>.

</div>

---
layout: default
---

# TPT — La idea

<div class="text-lg mt-6">

**Una tabla por clase**, tal cual el diagrama: `Persona`, `Funcionario`, `Docente` y `Estudiante`.

- La tabla padre guarda los **atributos comunes**.
- Cada tabla hija guarda **solo sus atributos propios**.
- El `Id` de la hija es **PK y FK a la vez**: apunta a la fila de `Persona`.
- **No hay discriminador**: el tipo se deduce de en qué tabla hija aparece el `Id`.

</div>

<div class="grid grid-cols-2 gap-6 mt-8 text-sm">

<div class="p-3 rounded bg-emerald-500/10 border border-emerald-500/30">

✅ Esquema **normalizado**, sin `NULL`. El más fiel al diagrama de clases.

</div>

<div class="p-3 rounded bg-red-500/10 border border-red-500/30">

❌ Toda consulta necesita **JOINs**. Insertar un `Docente` = 2 INSERTs.

</div>

</div>

---
layout: section
---

# TPC
## Table Per Concrete type

---
layout: default
---

# TPC — Cómo quedarían las tablas

<div class="flex items-center justify-center h-3/5 mt-2">

<img src="/TPC.png" class="max-h-full max-w-full object-contain rounded bg-white p-2 shadow" />

</div>

<div class="mt-4 p-3 rounded bg-emerald-500/10 border border-emerald-500/30 text-xs">

<b>3 tablas y ninguna <code>Persona</code>.</b> Las columnas comunes (<code>nombre</code>, <code>apellido</code>, <code>fechaNacimiento</code>, <code>email</code>) se repiten en las tres. Un <code>Docente</code> se trae de una sola tabla, sin JOIN.

</div>

---
layout: default
---

# TPC — La idea

<div class="text-lg mt-6">

**Una tabla por clase concreta**: `Funcionario`, `Docente` y `Estudiante`.

- `Persona` es abstracta → **no tiene tabla**.
- Cada tabla repite **todas** las columnas: las comunes y las propias.
- Tampoco hay discriminador.
- Los `Id` no se repiten entre tablas: EF usa una **secuencia compartida**.

</div>

<div class="grid grid-cols-2 gap-6 mt-8 text-sm">

<div class="p-3 rounded bg-emerald-500/10 border border-emerald-500/30">

✅ Consultar **un tipo concreto** no requiere JOINs. Sin `NULL`.

</div>

<div class="p-3 rounded bg-red-500/10 border border-red-500/30">

❌ Columnas comunes **duplicadas**. Traer todas las personas = `UNION` de las 3 tablas.

</div>

</div>

---
layout: section
---

# TPH
## Table Per Hierarchy

---
layout: default
---

# TPH — Cómo quedaría la tabla

<div class="flex items-center justify-center h-3/5 mt-2">

<img src="/TPH.png" class="max-h-full max-w-full object-contain rounded bg-white p-2 shadow" />

</div>

<div class="mt-4 p-3 rounded bg-cyan-500/10 border border-cyan-500/30 text-xs">

<b>1 sola tabla</b> con todas las columnas de la jerarquía + <code>discriminator</code>. Nunca hay JOIN, pero las columnas de las hijas quedan en <code>NULL</code> para las filas que no corresponden.

</div>

<!--
Pregunta típica de parcial: "mapeá esta jerarquía con X estrategia y dibujá las tablas".
-->

---
layout: default
---

# TPH — La idea

<div class="text-lg mt-6">

**Una sola tabla** `Persona` con todas las columnas de toda la jerarquía.

- Guarda los atributos comunes **y** los de todas las hijas.
- Aparece una **columna discriminadora** que indica de qué tipo es cada fila.
- Las columnas de las hijas tienen que ser **nullables**.
- Es la estrategia **por defecto** de EF Core.

</div>

<div class="grid grid-cols-2 gap-6 mt-8 text-sm">

<div class="p-3 rounded bg-emerald-500/10 border border-emerald-500/30">

✅ **Sin JOINs**: las consultas más rápidas. Esquema simple.

</div>

<div class="p-3 rounded bg-red-500/10 border border-red-500/30">

❌ Muchos `NULL` y se pierde la integridad (`NOT NULL`). La tabla crece a lo ancho.

</div>

</div>
