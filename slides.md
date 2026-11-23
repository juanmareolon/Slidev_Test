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

# TPT — La idea

<div class="text-lg mt-6">

**Una tabla por clase**, tal cual el diagrama: `Personas`, `Funcionarios`, `Docentes` y `Estudiantes`.

- La tabla padre guarda los **atributos comunes**.
- Cada tabla hija guarda **solo sus atributos propios**.
- El `Id` de la hija es **PK y FK a la vez**: apunta a la fila de `Personas`.
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
layout: default
---

# TPT — Cómo quedarían las tablas

<div class="grid grid-cols-2 gap-6 mt-4">

<div>

<div class="tname">Personas <span class="opacity-60">(datos comunes)</span></div>

<table>
<thead><tr><th>Id 🔑</th><th>Nombre</th><th>Apellido</th><th>FechaNac</th><th>Email</th></tr></thead>
<tbody>
<tr><td>1</td><td>Ana</td><td>Pérez</td><td>1990</td><td>ana@uni.edu</td></tr>
<tr><td>2</td><td>Luis</td><td>Gómez</td><td>1982</td><td>luis@uni.edu</td></tr>
<tr><td>3</td><td>Sofía</td><td>Díaz</td><td>2004</td><td>sofi@uni.edu</td></tr>
</tbody>
</table>

<div class="tname mt-4">Funcionarios</div>
<table>
<thead><tr><th class="fk">Id 🔑/FK</th><th>Sector</th><th>TipoContrato</th></tr></thead>
<tbody><tr><td class="fk">1</td><td>RRHH</td><td>Permanente</td></tr></tbody>
</table>

</div>

<div>

<div class="tname">Docentes</div>
<table>
<thead><tr><th class="fk">Id 🔑/FK</th><th>Catedra</th><th>Materia</th></tr></thead>
<tbody><tr><td class="fk">2</td><td>Informática</td><td>DA2</td></tr></tbody>
</table>

<div class="tname mt-4">Estudiantes</div>
<table>
<thead><tr><th class="fk">Id 🔑/FK</th><th>Carrera</th><th>AnioIngreso</th></tr></thead>
<tbody><tr><td class="fk">3</td><td>Ingeniería</td><td>2023</td></tr></tbody>
</table>

<div class="mt-6 p-3 rounded bg-amber-500/10 border border-amber-500/30 text-xs">

<b>4 tablas.</b> Para traer un <code>Docente</code> completo hay que hacer JOIN entre <code>Personas</code> y <code>Docentes</code>.

</div>

</div>

</div>

<style>
table { width: 100%; border-collapse: collapse; font-size: 0.7rem; }
th, td { border: 1px solid rgba(128,128,128,.45); padding: 3px 6px; text-align: left; white-space: nowrap; }
th { background: rgba(56,189,248,.18); font-weight: 600; }
.fk { background: rgba(251,191,36,.22); }
.tname { font-size: 0.8rem; font-weight: 600; margin-bottom: 3px; }
</style>

---
layout: section
---

# TPC
## Table Per Concrete type

---
layout: default
---

# TPC — La idea

<div class="text-lg mt-6">

**Una tabla por clase concreta**: `Funcionarios`, `Docentes` y `Estudiantes`.

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

❌ Columnas comunes **duplicadas**. Traer todas las `Personas` = `UNION` de las 3 tablas.

</div>

</div>

---
layout: default
---

# TPC — Cómo quedarían las tablas

<div class="text-sm mb-3 opacity-70">No existe tabla <code>Personas</code>: las columnas comunes (en azul) se repiten en cada tabla.</div>

<div class="tname">Funcionarios</div>
<table>
<thead><tr><th class="com">Id</th><th class="com">Nombre</th><th class="com">Apellido</th><th class="com">FechaNac</th><th class="com">Email</th><th>Sector</th><th>TipoContrato</th></tr></thead>
<tbody><tr><td class="com">1</td><td class="com">Ana</td><td class="com">Pérez</td><td class="com">1990</td><td class="com">ana@uni.edu</td><td>RRHH</td><td>Permanente</td></tr></tbody>
</table>

<div class="tname mt-4">Docentes</div>
<table>
<thead><tr><th class="com">Id</th><th class="com">Nombre</th><th class="com">Apellido</th><th class="com">FechaNac</th><th class="com">Email</th><th>Catedra</th><th>Materia</th></tr></thead>
<tbody><tr><td class="com">2</td><td class="com">Luis</td><td class="com">Gómez</td><td class="com">1982</td><td class="com">luis@uni.edu</td><td>Informática</td><td>DA2</td></tr></tbody>
</table>

<div class="tname mt-4">Estudiantes</div>
<table>
<thead><tr><th class="com">Id</th><th class="com">Nombre</th><th class="com">Apellido</th><th class="com">FechaNac</th><th class="com">Email</th><th>Carrera</th><th>AnioIngreso</th></tr></thead>
<tbody><tr><td class="com">3</td><td class="com">Sofía</td><td class="com">Díaz</td><td class="com">2004</td><td class="com">sofi@uni.edu</td><td>Ingeniería</td><td>2023</td></tr></tbody>
</table>

<div class="mt-6 p-3 rounded bg-emerald-500/10 border border-emerald-500/30 text-xs">

<b>3 tablas.</b> Un <code>Docente</code> se trae de una sola tabla, sin JOIN.

</div>

<style>
table { width: 100%; border-collapse: collapse; font-size: 0.7rem; }
th, td { border: 1px solid rgba(128,128,128,.45); padding: 3px 6px; text-align: left; white-space: nowrap; }
th { background: rgba(148,163,184,.18); font-weight: 600; }
.com { background: rgba(56,189,248,.20); }
.tname { font-size: 0.8rem; font-weight: 600; margin-bottom: 3px; }
</style>

---
layout: section
---

# TPH
## Table Per Hierarchy

---
layout: default
---

# TPH — La idea

<div class="text-lg mt-6">

**Una sola tabla** `Personas` con todas las columnas de toda la jerarquía.

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

---
layout: default
---

# TPH — Cómo quedaría la tabla

<div class="text-sm mb-3 opacity-70">Una única tabla <code>Personas</code>. En amarillo, la columna discriminadora.</div>

<table>
<thead>
<tr>
<th>Id</th><th class="disc">TipoPersona</th><th>Nombre</th><th>Apellido</th><th>FechaNac</th><th>Email</th>
<th>Sector</th><th>TipoContrato</th><th>Catedra</th><th>Materia</th><th>Carrera</th><th>AnioIngreso</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td><td class="disc">Funcionario</td><td>Ana</td><td>Pérez</td><td>1990</td><td>ana@uni.edu</td>
<td>RRHH</td><td>Permanente</td><td class="nul">NULL</td><td class="nul">NULL</td><td class="nul">NULL</td><td class="nul">NULL</td>
</tr>
<tr>
<td>2</td><td class="disc">Docente</td><td>Luis</td><td>Gómez</td><td>1982</td><td>luis@uni.edu</td>
<td class="nul">NULL</td><td class="nul">NULL</td><td>Informática</td><td>DA2</td><td class="nul">NULL</td><td class="nul">NULL</td>
</tr>
<tr>
<td>3</td><td class="disc">Estudiante</td><td>Sofía</td><td>Díaz</td><td>2004</td><td>sofi@uni.edu</td>
<td class="nul">NULL</td><td class="nul">NULL</td><td class="nul">NULL</td><td class="nul">NULL</td><td>Ingeniería</td><td>2023</td>
</tr>
</tbody>
</table>

<div class="mt-6 p-3 rounded bg-cyan-500/10 border border-cyan-500/30 text-xs">

<b>1 tabla.</b> Nunca hay JOIN, pero más de la mitad de las celdas quedan en <code>NULL</code>.

</div>

<style>
table { width: 100%; border-collapse: collapse; font-size: 0.65rem; }
th, td { border: 1px solid rgba(128,128,128,.45); padding: 3px 5px; text-align: left; white-space: nowrap; }
th { background: rgba(56,189,248,.18); font-weight: 600; }
.disc { background: rgba(251,191,36,.22); }
.nul { opacity: .4; font-style: italic; }
</style>

<!--
Pregunta típica de parcial: "mapeá esta jerarquía con X estrategia y dibujá las tablas".
-->
