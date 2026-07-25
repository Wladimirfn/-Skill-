# IDUBOX Skill 

```
██╗██████╗ ██╗   ██╗██████╗  ██████╗ ██╗    ██╗    
██║██   ██╗██║   ██║██╔══██╗██╔═══██╗ ██║  ██╔╝
██║██   ██ ██║   ██║██████╔╝██║   ██║  █████╔╝ 
██║██   ██╗██║   ██║██╔══██╗██║   ██║ ██╔══██╗ 
██║██████╔╝╚██████╔╝██████╔╝╚██████╔╝██║    ██╗
╚═╝╚═════╝  ╚═════╝ ╚═════╝  ╚═════╝ ╚═╝    ╚═╝    
```

**IDUBOX Skill** es mi repositorio de skill personales, pensado para el ecosistema **IDUBOX** y para mis proyectos de RCM y mantenimiento industrial.  
Aquí voy dejando, de forma ordenada, lo que voy aprendiendo y las buenas prácticas que voy definiendo.

La idea es simple:

- Cada vez que aprenda algo útil, lo convierto en una skill.
- Cada skill busca ser práctica, aplicable y probada en casos reales.
- Con el tiempo, este repo debería convertirse en mi propia “documentación viva”.

## ¿Qué vas a encontrar aquí?

- Buenas prácticas de bases de datos (PostgreSQL / Supabase, RLS, multi-tenant, migraciones, etc.).
- Guías y notas pensadas para sistemas de mantención, **RCM** y software industrial.
- Apuntes técnicos que uso en mi día a día como desarrollador y responsable de sistemas en el entorno IDUBOX.

## Objetivo del repositorio

No es un curso ni un libro, es mi propio mapa de conocimientos.  
Lo uso para:

- No repetir errores en proyectos futuros (especialmente en IDUBOX y derivados).
- Tener una referencia rápida de decisiones técnicas importantes.
- Ver cómo evoluciona mi forma de trabajar con el tiempo.

## Estructura (en evolución)

Por ahora, iré agregando carpetas y archivos por tema, por ejemplo:

- `buenas-practicas-bd/` – Buenas prácticas de bases de datos y Supabase.
- `Normalizacion DB/` – Notas de normalización de bases de datos.
- `review-ambiguous-start/` – Gentle AI 2.1.11: qué hacer cuando `review status` devuelve `applicability: ambiguous` / `action: select_lineage`. Evita seleccionar una lineage ajena o bypassear el guard cuando la solución real es simplemente correr `review start`.
- `review-capture-helper/` – Gentle AI 2.1.11: workaround para cuando `review capture-result` falla con `decode reviewer result: invalid character ...` (el reviewer devuelve JSON envuelto en prosa/fence markdown). Recupera el `.raw` preservado por identidad exacta (lineage+lens+order, nunca por fecha) y lo reenvía por el facade nativo.
- Más skills se irán sumando a medida que vaya aprendiendo y formalizando lo que hago.

---

Seguiré actualizando **IDUBOX Skill** a medida que aprenda cosas nuevas, refine mis procesos y descubra mejores formas de trabajar dentro y fuera de IDUBOX.
