Vistas del turista para "Sucre Turístico" (Golfo de Morrosquillo): Inicio, Destinos, Detalle de
destino, Alojamiento, Gastronomía, Experiencias, Eventos y Buscador.

Ya está conectado al backend real (`gateway` + `destinos-service` + `alojamiento-service` +
`gastronomia-service`, con MongoDB). Experiencias, Eventos y el buscador con Elasticsearch
todavía no tienen microservicio, así que siguen mostrando datos de ejemplo hasta la próxima
entrega.

## Requisitos

- Node.js 18 o superior
- El backend (`Sucre_Turistico_Backend`) corriendo — con Docker Compose o local con MongoDB

## 1. Levanta primero el backend

Desde la carpeta del backend:

```bash
docker compose up --build
```

o, sin Docker, en 4 terminales (`gateway`, `destinos-service`, `alojamiento-service`,
`gastronomia-service`), con MongoDB corriendo en `mongodb://localhost:27017`:

```bash
cd <servicio>
cp .env.example .env
npm install
npm start
```

Verifica que responde: `curl http://localhost:4000/health`

## 2. Levanta el frontend

```bash
cd sucre-turistico
npm install
cp .env.example .env
npm run dev
```

Abre http://localhost:5173.

Si el backend no está corriendo, el frontend no se rompe: cada página cae de vuelta a datos de
ejemplo (verás una advertencia en la consola del navegador) para que puedas seguir viendo el
diseño mientras levantas los servicios.

## Cómo está organizado

```
src/
  api/
    client.js        Instancia de Axios (baseURL = VITE_API_URL, interceptores)
    endpoints.js      Una función por recurso + adaptadores de los campos de MongoDB
  data/
    mockData.js        Datos de respaldo (destinos/alojamiento/gastronomia) y de ejemplo
                        (experiencias/eventos, que aún no tienen backend)
  utils/
    colors.js           Genera colores estables para las ilustraciones a partir del id
    format.js            Formato de precios en COP y de fechas
  components/           Piezas reutilizables (tarjetas, filas, filtros, topbar)
  pages/                  Una página por vista
  styles/                  CSS con las variables de diseño
```

## Campos reales del backend (importante para el panel admin)

| Recurso      | Endpoint (gateway)         | Campos en MongoDB                                         |
|--------------|------------------------------|-------------------------------------------------------------|
| Destinos     | `/api/destinos`               | `nombre`, `municipio`, `categoria`, `descripcion`, `imagenUrl`, `activo` |
| Alojamiento  | `/api/alojamiento` (singular) | `nombre`, `municipio`, `tipo`, `precioNoche`, `descripcion`, `activo` |
| Gastronomía  | `/api/gastronomia`             | `nombre`, `municipio`, `tipoCocina`, `rangoPrecio`, `descripcion`, `activo` |

`endpoints.js` traduce estos nombres (`categoria` → `tipo`, `tipoCocina` → `categoria`, etc.) a
los que ya usan los componentes, así que no hace falta tocar ninguna vista si el modelo de datos
cambia un poco — solo ajustas la función `adapt...` correspondiente.

Los filtros (facets) de Destinos, Alojamiento y Gastronomía ya no son una lista fija: se arman
con los valores que realmente existan en la base de datos, porque `categoria`, `tipo` y
`tipoCocina` son campos de texto libre que se llenan desde el panel admin.

## Agregar hoteles e imágenes

Ya existe un formulario funcional en **/admin** (botón "Administrador" en la barra superior) para
agregar y eliminar alojamiento. Como Mongo solo guarda texto, el campo "URL de la imagen" espera
un enlace público a la foto (súbela primero a algo como Cloudinary, Imgur o un bucket de S3, y
pega el enlace ahí) — este proyecto todavía no sube archivos por sí mismo.

Nota: para que el campo de imagen se guarde, el `alojamiento-service` necesita el campo
`imagenUrl` en su modelo. Si tu backend todavía no lo tiene, reemplaza
`alojamiento-service/src/models/alojamiento.model.js` por la versión que viene junto a este
paquete (o agrégale la línea `imagenUrl: { type: String, required: false },`).

## Siguientes pasos sugeridos

- Construir `experiencias-service` y `eventos-service` (misma estructura que los tres actuales)
  y reemplazar los mocks correspondientes en `src/api/endpoints.js`.
- Construir `buscador-service` con Elasticsearch y cambiar `search()`/`getAllSearchable()` en
  `endpoints.js` para llamar a `GET /api/buscador` en vez de combinar los datos en el frontend.
- Conectar el login de administrador (`src/pages/Admin.jsx`) contra un endpoint de autenticación
  (todavía no existe en el backend, según su propio README).
- Añadir los formularios CRUD del panel admin (POST/PUT/DELETE ya existen en los 3 servicios).

## Build para producción

```bash
npm run build
npm run preview
```

El resultado queda en `dist/`, listo para desplegar en Vercel o cualquier hosting estático.
