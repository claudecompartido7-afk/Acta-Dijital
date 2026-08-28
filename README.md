# SIGA — Sistema Integral de Actas

Registro digital de actas de reunión de la OGPL — UNMSM.
Proyecto de Google Apps Script (Web App) + Google Sheets + Google Drive.

## Archivos

```
siga/Codigo.gs      Backend: autenticación, generación del PDF, dashboard
siga/Index.html     Frontend: login, formulario de acta, panel gerencial, admin
siga/Config.gs      Constante LOGO_UNMSM (logo institucional en Base64)
```

## Despliegue

1. Crear un proyecto en [script.google.com](https://script.google.com).
2. Crear los tres archivos con estos nombres exactos: `Codigo.gs`, `Config.gs`
   y `Index.html` (el `doGet` carga la plantilla por el nombre `Index`).
3. Ajustar en `Codigo.gs` las constantes `CONFIG.FOLDER_ID` (carpeta de Drive
   donde se guardan los PDF) y `CONFIG.SPREADSHEET_ID` (base de datos).
4. Ejecutar una vez `inicializarSistema()` — crea las hojas `Usuarios`,
   `Actas`, `Asistentes` y `Acuerdos` con sus encabezados.
5. Ejecutar una vez `crearUsuarioAdmin()` — crea `admin@unmsm.edu.pe` con
   contraseña `admin123`. **Cambiarla después del primer ingreso.**
6. Publicar como aplicación web (Implementar → Nueva implementación).

## Estructura de la base de datos

| Hoja | Columnas |
|---|---|
| Usuarios | Email, PasswordHash, Nombre, Rol, UnidadOficina, FechaRegistro |
| Actas | ID_Acta, Tema, Modalidad, Lugar, Fecha, HoraInicio, HoraFin, URL_PDF, RegistradoPor, Timestamp, CantidadAsistentes, CantidadFotos |
| Asistentes | ID_Acta, Nombres, Apellidos, Cargo, Unidad, TieneFirma, Timestamp |
| Acuerdos | ID_Acuerdo, ID_Acta, Acuerdo, Responsable, Plazo, UnidadPlazo, FechaLimite, Estado, FechaCumplimiento, DiasRestantes, Indicador, Timestamp |

## Codificación del acta

`Acta N° 001-2026-OR-OGPL/UNMSM`

El correlativo sale de `getLastRow()` de la hoja `Actas`, protegido con
`LockService` para evitar duplicados si dos usuarios guardan a la vez.

## Cálculo de plazos

`calcularFechaLimite()` soporta `horas`, `días hábiles` y `meses`.
En días hábiles salta sábados, domingos y los feriados de `FERIADOS_PERU`
(cargados para 2026).

## Fórmula del panel gerencial

```
% Cumplimiento = [(Cumplidos × 1) + (En Plazo × 0.75) + (Por Vencer × 0.5)] / Total × 100
```

## Acuerdos vs. Compromisos

El formulario distingue dos listas y ambas se guardan en la hoja `Acuerdos`:

| | Contenido | ID | Estado inicial | Indicador |
|---|---|---|---|---|
| **Acuerdo** | Decisión sin fecha límite | `..._AC001` | `Cumplido` | `ACUERDO` |
| **Compromiso** | Tarea con plazo y responsable | `..._CO002` | `Pendiente` | `PENDIENTE` |

El correlativo es continuo entre ambas listas dentro de la misma acta.

## Pendientes conocidos

- **`Config.gs` tiene un logo de marcador de posición** (PNG transparente de
  1×1). Falta pegar el Base64 real de `LOGO_UNMSM`; las instrucciones están
  en los comentarios del propio archivo.

- **Los acuerdos sin plazo se guardan con estado `Cumplido`.** Es una decisión
  de diseño para que no aparezcan como pendientes, pero tiene un efecto
  secundario: entran en el KPI «Acuerdos Cumplidos» y elevan el porcentaje de
  cumplimiento global, que mide el avance de los compromisos. Si prefieres que
  queden fuera del cálculo, hay que tratarlos como informativos en
  `obtenerAcuerdos()` y en `renderizarDashboard()`.
