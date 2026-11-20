# NutriComunidad – Frontend (Vite + JavaScript Vanilla)

Frontend multipágina de NutriComunidad, conectado al backend REST (Spring Boot, repositorio
[APINT](https://github.com/DiegoNIX75/APINT), rama `develop`). Diseño basado en el Figma
*WireFrameNutricomunidad*. Sin frameworks: HTML, CSS y JS con módulos ES.

## Ejecutar en local

Requisitos: Node 20+ y el backend corriendo (por defecto en `http://localhost:8080`).

```bash
npm install
cp .env.example .env.local      # Windows: copy .env.example .env.local
# en .env.local: VITE_API_URL=http://localhost:8080
npm run dev                     # http://localhost:5173
```

Otros comandos: `npm run build` (genera `dist/`) y `npm run preview` (sirve el build).

El backend ya permite `http://localhost:*` por CORS. Para probar sin Neon puedes levantarlo con
PostgreSQL embebido: en el repositorio del backend, `./gradlew bootTestRun`.

## Estructura

```
index.html                 Portada pública
login.html                 Inicio de sesión
Pages/
  registro.html            Registro (Beneficiario, Comercio u Organización)
  admin/                   usuarios, comercios, organizaciones, beneficiarios, recetas, alimentos
  beneficiario/            menu, perfil, solicitud, donaciones-recibidas, donaciones-solicitadas
  comercio/                mi-comercio, productos
  organizacion/            mi-organizacion, solicitudes, donaciones
src/
  css/                     normalize, global (variables y componentes), portada, acceso, panel
  img/                     emblema, íconos y recetas/ (imágenes de recetas, ver abajo)
  partials/                encabezado-panel, barra-lateral, encabezado-pagina
  js/
    config.js              URL del backend (VITE_API_URL)
    api/                   client.js (fetch central) + un módulo por recurso
    auth/                  session.js (localStorage/JWT), guard.js (protección), roles.js
    layout/panel.js        Monta encabezado + menú lateral + título y protege la página
    ui/                    DOM seguro, formato, avisos/confirmaciones, estados, tablas, modales,
                           formularios, campos de entidades, recetas
    pages/                 Lógica de cada página (misma ruta que su HTML)
```

## Autenticación y roles

- `POST /api/auth/login` devuelve `{ token, usuario }`. El JWT (HMAC256) contiene `sub` (id),
  `nombre`, `rol` y `exp`. Se guarda en `localStorage` (`nutricomunidad.token` y
  `nutricomunidad.usuario`).
- `api/client.js` envía `Authorization: Bearer <token>` en cada petición. Si el backend responde
  401, borra la sesión y lleva al login con «Tu sesión expiró».
- `auth/guard.js` protege cada página privada: sin sesión vigente (o con `exp` vencido) redirige a
  `login.html`. Si el rol no corresponde, envía al usuario a la página de inicio de su rol. Es sólo
  navegación: los permisos reales los valida el backend (403).
- Redirección por rol tras el login (`auth/roles.js`):

| Rol (backend) | Página de inicio |
|---|---|
| Administrador | `Pages/admin/usuarios.html` |
| Beneficiario | `Pages/beneficiario/menu.html` |
| Comercio | `Pages/comercio/mi-comercio.html` |
| Organizacion | `Pages/organizacion/mi-organizacion.html` |

## Páginas, funciones y endpoints

| Página | Funciones | Endpoints |
|---|---|---|
| Portada | Presentación, acceso al login; con sesión muestra «Ir a mi panel» | — |
| Login | Validación, carga, errores, evita doble envío, redirección por rol | `POST /api/auth/login` |
| Registro | Tipo de cuenta + datos del rol; cuenta y entidad se crean juntas | `POST /api/auth/registro` |
| Admin · Usuarios | Listar (Mostrar listado), filtros estado/rol/búsqueda, alta, baja/reactivar, cambiar rol, eliminar, imprimir | `GET/POST /api/usuarios`, `PATCH /api/usuarios/{id}/estado`, `PATCH …/rol`, `DELETE …`, `GET /api/roles` |
| Admin · Comercios | Búsqueda, tipo, ficha con donaciones y cantidad donada, editar, eliminar | `GET/PUT/DELETE /api/comercios`, `GET /api/donaciones?idComercio=`, `GET /api/donaciones/{id}` |
| Admin · Organizaciones | Búsqueda, ficha con solicitudes, atendidas y existencias, editar, eliminar | `GET/PUT/DELETE /api/organizaciones`, `GET …/{id}/solicitudes`, `GET /api/inventarios?idOrganizacion=` |
| Admin · Beneficiarios | Búsqueda, ficha con apoyos, solicitudes, pendientes y últimos movimientos, editar, eliminar | `GET/PUT/DELETE /api/beneficiarios`, `GET …/{id}/solicitudes`, `GET /api/entregas` |
| Admin · Recetas | CRUD de recetas e ingredientes, imagen reservada | `GET/POST/PUT/DELETE /api/recetas`, `POST/DELETE /api/recetas/{id}/ingredientes` |
| Admin · Alimentos | CRUD de alimentos, categorías y vista «por lote» | `/api/alimentos`, `/api/categorias`, `GET /api/lotes`, `GET /api/inventarios/{id}`, `GET /api/donaciones/{id}` |
| Beneficiario · Menú | Recetas comunes y «según tus preferencias» (excluye alergias), filtros, carrusel, detalle | `GET /api/recetas`, `GET /api/recetas?idAlimento=`, `GET /api/recetas/{id}`, `GET /api/beneficiarios/mio`, `GET …/{id}/alergias` |
| Beneficiario · Mi perfil | Datos de cuenta y familia, contraseña, preferencias, alergias y restricciones | `GET/PUT /api/auth/yo`, `PUT /api/auth/yo/contrasena`, `GET/PUT /api/beneficiarios/mio`, `/api/preferencias`, `…/{id}/alergias` |
| Beneficiario · Enviar solicitud | Organización, motivo y prioridad; fecha e integrantes automáticos | `GET /api/organizaciones`, `POST /api/solicitudes` |
| Beneficiario · Donaciones solicitadas | Historial con filtros por estado y conteos | `GET /api/solicitudes` (sólo las propias) |
| Beneficiario · Donaciones recibidas | Entregas de sus solicitudes | `GET /api/entregas`, `GET /api/solicitudes` |
| Comercio · Mi comercio | Ficha, miembro desde, estadísticas, editar datos | `GET/PUT /api/comercios/mio`, `GET /api/auth/yo`, `GET /api/donaciones` |
| Comercio · Productos a donar | Armar donación con nutrición y totales, enviar, historial | `GET /api/alimentos`, `POST /api/donaciones`, `GET /api/donaciones/{id}` |
| Organización · Mi organización | Ficha, familias atendidas, solicitudes del mes, comercios vinculados, editar | `GET/PUT /api/organizaciones/mio`, `GET /api/solicitudes`, `GET /api/inventarios` |
| Organización · Solicitudes | Filtro por prioridad, ficha, despachar (crea entrega), rechazar, marcar entregada, imprimir | `GET /api/solicitudes`, `GET /api/beneficiarios`, `GET …/{id}/entregas`, `POST /api/entregas`, `PATCH /api/entregas/{id}/estado`, `PATCH /api/solicitudes/{id}/estado` |
| Organización · Donaciones | Registrar donaciones en el inventario, existencias, lotes | `GET /api/donaciones`, `GET/POST/PUT /api/inventarios`, `GET …/{id}/lotes`, `POST /api/lotes` |

Todas las páginas muestran estado de carga, estado vacío y errores comprensibles (incluye reintentar).
Las acciones destructivas piden confirmación.

## Funcionalidades del Figma no implementadas (sin soporte en el backend)

| Pantalla / elemento | Motivo |
|---|---|
| Beneficiario · Proponer receta, y origen «Propuesta» en Admin · Recetas | No existe el concepto de propuesta ni su estado de revisión |
| Comercio · Estadísticas (merma, aprovechado) y métricas «Iniciales / Desperdicio» en Admin · Comercios | La BD no guarda productos iniciales ni merma |
| Filtros Desayuno / Comida / Cena del menú | Las recetas no tienen tipo de comida |
| «Observaciones» al enviar una solicitud | `SOLICITUDES` no tiene ese campo |
| Estado por producto («Por vencer», «Disponible») en Productos a donar | El detalle de donación no tiene estado ni fecha de caducidad |
| Campo «Usuario» del registro y del perfil, y «Usuario o correo» en el login | No existe nombre de usuario; el login es por correo |
| «Vista previa del prototipo · entrar como…» del login | Era un atajo del prototipo; requeriría credenciales en el frontend |
| «Asignar donación» como relación | `ENTREGAS` no se relaciona con el inventario; la asignación se guarda como texto en las observaciones de la entrega |

## Imágenes

**Recetas.** Coloca las imágenes en `src/img/recetas/` con el nombre `receta-{idReceta}.webp`
(también acepta `.jpg`, `.jpeg` o `.png`). Por ejemplo, la receta con id 3 se llama `receta-3.webp`;
el id aparece en Admin · Recetas o en `GET /api/recetas`. No hay que tocar código: `src/js/ui/recetas.js`
las detecta con `import.meta.glob` y sustituye el espacio reservado (`.marcador-imagen`); si una
receta no tiene archivo, se mantiene el espacio vacío. Formato recomendado: WebP, 800×500 px
(proporción 16:10, la de la tarjeta), menos de 150 KB.

**Otras imágenes reservadas** (opcionales): la imagen principal de la portada y las fotos de los
creadores están en `index.html` como `<div class="marcador-imagen">`. Para usarlas, copia el archivo en
`src/img/` y cambia el `div` por `<img src="/src/img/archivo.webp" alt="…">` (dentro del mismo
contenedor, que ya tiene tamaño y bordes).

## Despliegue: Vercel (frontend) + Render (backend) + Neon (BD)

```
Frontend (Vercel) ──HTTP/REST + JWT──▶ Backend (Render) ──JDBC──▶ PostgreSQL (Neon)
```

### 1. Neon
1. Ejecuta `sql/schema.sql` del backend en el SQL Editor (es idempotente; también agrega `id_usuario`
   si la base ya existía).
2. En «Connect», copia el host **sin `-pooler`**, la base, el usuario y la contraseña.

### 2. Backend en Render
Render no tiene entorno Java nativo, así que el backend necesita un `Dockerfile` en la raíz de su
repositorio. Todavía no existe y no lo probé, porque este equipo no tiene Docker. Uno compatible
con el proyecto:

```dockerfile
FROM eclipse-temurin:17-jdk AS build
WORKDIR /app
COPY . .
RUN chmod +x gradlew && ./gradlew bootJar --no-daemon -x test

FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/build/libs/*-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

1. Render → New → Web Service → conecta el repositorio del backend (rama `develop` o `main`) →
   Runtime **Docker**.
2. Variables de entorno: `DB_URL=jdbc:postgresql://<host>.neon.tech/<base>?sslmode=require`,
   `DB_USER`, `DB_PASSWORD`, `JWT_SECRET` (32+ caracteres aleatorios), `JWT_EXPIRACION_HORAS=8`,
   `CORS_ORIGENES=https://<tu-proyecto>.vercel.app` y, opcionalmente, `ADMIN_CORREO` y
   `ADMIN_PASSWORD`. Render asigna `PORT` solo; el backend ya lo lee.
3. Verifica `https://<servicio>.onrender.com/api/health/db`, que debe decir «Conexión a la base de
   datos correcta». En el plan gratuito, el primer acceso tras un rato inactivo tarda unos segundos.

### 3. Frontend en Vercel
1. Vercel → Add New → Project → importa este repositorio. Framework: **Vite**; build `npm run build`;
   salida `dist` (ya definido en `vercel.json`).
2. Variable de entorno: `VITE_API_URL=https://<servicio>.onrender.com`, sin `/` final. Se lee **en el
   build**, así que si la cambias debes volver a desplegar (Deployments → Redeploy).
3. Las rutas distinguen mayúsculas: usa `/Pages/...` tal cual. Cada página es un HTML real, así que
   no hace falta reescritura de rutas.
4. Cada `git push` a la rama conectada publica una nueva versión.

### 4. Conexión final y verificación
1. Agrega el dominio de Vercel a `CORS_ORIGENES` en Render, separado por coma si hay varios, y
   reinicia el servicio.
2. Abre el sitio de Vercel → Iniciar sesión. En DevTools → Network, la petición `login` debe ir a
   `onrender.com` y responder 200.
3. En las siguientes peticiones revisa que el header `Authorization: Bearer …` esté presente.
4. Si aparece un error de CORS en consola, el origen no está en `CORS_ORIGENES`, o tiene `/` final
   o un protocolo distinto.
5. Crea un registro (por ejemplo, una categoría como admin) y confírmalo en Neon (Tables).

## Checklist de pruebas

- [ ] `npm run build` termina sin errores.
- [ ] La portada abre sin sesión y «Iniciar sesión» lleva a `login.html`.
- [ ] Un login con credenciales incorrectas muestra «Correo o contraseña incorrectos».
- [ ] El login de cada rol redirige a su panel y `localStorage` contiene `nutricomunidad.token`.
- [ ] Una página de otro rol redirige al panel propio; una página privada sin sesión lleva al login.
- [ ] «Cerrar sesión» borra el token.
- [ ] El registro de los tres roles funciona; un correo repetido muestra el aviso de duplicado.
- [ ] Comercio: agrega productos, envía la donación y la ve en el historial.
- [ ] Organización: registra la donación en su inventario, agrega un lote, despacha una solicitud y
      la marca como entregada.
- [ ] Beneficiario: guarda preferencias y alergias, el menú excluye recetas con alérgenos, envía una
      solicitud y ve la entrega en «Donaciones recibidas».
- [ ] Admin: Usuarios (Mostrar listado, dar de baja, cambiar rol), Comercios, Organizaciones,
      Beneficiarios, Recetas e ingredientes, Alimentos y categorías.
- [ ] Vista móvil (375 px): menú lateral desplegable y sin desplazamiento horizontal.
