# The Lighthouse School · Actividad evaluativa de Química 10°

Aplicación fullstack (Next.js 16 + PostgreSQL + Drizzle ORM) con la actividad evaluativa **«Unidades físicas y químicas de concentración»** para grado 10°, más una **consola de control docente en vivo**.

![Escudo institucional](public/brand/escudo.jpg)

## Contenido de la actividad (45 minutos · 100 puntos)

| Sección | Contenido | Ítems | Puntos |
| --- | --- | --- | --- |
| 1. Laboratorio virtual | Tres simulaciones 3D interactivas con cantidades dadas en el protocolo (balanza, piseta, bureta con goteo, pipeta): el estudiante opera el material y registra las lecturas con sus unidades | 9 | 30 |
| 2. Química en contexto | Unidades físicas aplicadas a contaminantes (ppm, mg/L) y casos médicos (% m/v, mg/dL, % v/v, % m/m) | 6 | 30 |
| 3. Prueba Saber + abiertas | 5 preguntas de selección múltiple con única respuesta y 3 preguntas abiertas sobre aplicabilidad | 8 | 40 |

Las prácticas incluyen simulaciones manipulables con animaciones (chorro de la piseta, goteo de la bureta, agitación magnética, viraje del indicador), retroalimentación inmediata en la sección 1 y un **informe PDF** generado con `pdf-lib` (con modo de respaldo automático) para cargar en **Tareas de SINAI**.

## Reglas implementadas

- ✅ **Pantalla completa obligatoria** con registro de cada salida (y penalización automática desde la segunda salida).
- ✅ **Copiar, pegar, cortar, clic derecho, arrastrar y atajos** bloqueados; cada intento queda en la bitácora.
- ✅ **Avance secuencial**: una sección se habilita cuando todos los ítems de la anterior fueron enviados (validado también en el servidor).
- ✅ **Cronómetro de 45 minutos** con guardado automático de cada respuesta y del progreso general.
- ✅ **Reanudación con PIN docente `82032350727`** (configurable con `PIN_REANUDACION`). El PIN otorga tiempo adicional y conserva sección, respuestas y cronómetro.
- ✅ **Roles de misión** (analista de laboratorio, técnico ambiental, asistente clínico) elegidos al ingresar; contextualizan las secciones y se reportan en el PDF.
- ✅ **Informe PDF** con identificación, rol, puntajes, claves, respuestas, bitácora y pasos para SINAI. Se habilita al finalizar o al entregar parcialmente; descarga verificada con modo de respaldo.
- ✅ **Consola docente** (`/docente` y `public/consola-docente.html`): monitoreo en vivo, penalizaciones (poner/quitar), mensajes a la clase o individuales, tiempo extra, pausar/reabrir, calificar abiertas, sonómetro con micrófono y cierre global de la actividad.

## Versiones de un solo archivo (sin servidor)

| Archivo | Uso |
| --- | --- |
| `public/actividad-evaluativa.html` | Actividad evaluativa completa (roles, 3 secciones, simulaciones, cronómetro, PIN y **PDF generado en el navegador**). Se abre con doble clic. |
| `public/consola-docente.html` | Consola docente autónoma (monitoreo, penalizaciones, mensajes, sonómetro, tiempo extra, cierre global e informes PDF). |

Ambos incluyen las imágenes 3D incrustadas en base64 (no requieren internet), comparten el avance mediante
`localStorage` cuando se abren en el mismo navegador y también permiten **exportar/importar un respaldo JSON** para
trabajar entre equipos. Desde el servidor Next.js están disponibles en
`/actividad-evaluativa.html` y `/consola-docente.html`.

## Puesta en marcha

```bash
npm install
cp .env.example .env          # ajusta DATABASE_URL
npx drizzle-kit push          # crea las tablas
npm run dev                   # http://localhost:3000
```

Rutas:

- `/` — Ingreso del estudiante (crea o retoma la sesión) y reglas de la actividad.
- `/examen?sid=<id>` — Experiencia del estudiante (pantalla completa, 3 secciones, PDF).
- `/docente` — Consola de control docente en React.
- `/consola-docente.html` — Consola docente autónoma en HTML/JS (misma API).
- `/api/health` — Verificación de salud (base de datos incluida).

## Roles

| Rol | Enfoque |
| --- | --- |
| ⚗️ Analista de laboratorio | Precisión instrumental y cifras significativas |
| 🌊 Técnico de calidad ambiental | Normas ambientales y ppm |
| 🏥 Asistente clínico | Seguridad del paciente y conversión de unidades |

## Variables de entorno

| Variable | Descripción | Valor por defecto |
| --- | --- | --- |
| `DATABASE_URL` | Cadena de conexión PostgreSQL | `postgresql://postgres:postgres@127.0.0.1:5432/app_db` |
| `DOCENTE_CLAVE` | Clave de acceso a la consola docente | `LIGHTHOUSE-DOCENTE-2026` |
| `PIN_REANUDACION` | PIN entregado a los estudiantes para continuar | `82032350727` |

## Estructura

```
src/
├── app/
│   ├── api/
│   │   ├── docente/{panel,acciones}    # monitoreo y controles
│   │   ├── sesiones/                   # creación, estado, respuestas, reanudar
│   │   └── sesiones/[id]/reporte       # informe PDF
│   ├── docente/page.tsx                # consola docente
│   ├── examen/page.tsx                 # examen del estudiante
│   └── page.tsx                        # inicio / ingreso
├── components/
│   ├── Banner.tsx                      # escudo + nombre (siempre visible)
│   ├── docente/ConsolaDocente.tsx
│   └── examen/{ExamenApp,Secciones,Simulaciones,tipos}.tsx
├── db/{index,schema}.ts                # Drizzle + PostgreSQL
└── lib/{examen,calificacion,servidor}.ts
public/
├── brand/escudo.(png|jpg)              # escudo institucional
├── img/caso-{medico,ambiental}.jpg     # imágenes 3D de contexto
├── lab/render-*.jpg                    # renders 3D del material
├── consola-docente.html                # consola HTML autónoma (un solo archivo)
└── actividad-evaluativa.html           # actividad HTML autónoma (un solo archivo)
docs/ACTIVIDAD.md                       # guía docente, rúbricas y solucionario
```

## Despliegue en GitHub / producción

1. Crea el repositorio y sube el proyecto (`.env` está excluido por `.gitignore`).
2. Configura las variables de entorno en el proveedor (Vercel, Railway, Render…).
3. Usa PostgreSQL administrado y ejecuta `npx drizzle-kit push`.
4. El flujo de integración continua está en `.github/workflows/ci.yml` (typecheck + build).

> Nota de integridad: el navegador nunca recibe las claves de respuesta ni las explicaciones antes de finalizar; la calificación se realiza en el servidor (`src/lib/calificacion.ts`).
