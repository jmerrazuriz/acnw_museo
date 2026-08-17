# Cuaderno de Isla

Registro y catálogo de las 200 criaturas de *Animal Crossing: New Horizons* (80 peces, 80 bichos, 40 criaturas marinas), pensado para usarse en el teléfono mientras juegas.

No necesita servidor, ni base de datos, ni cuenta. Son archivos estáticos: los subes a GitHub Pages y ya funciona.

## Qué hace

**Ahora** — Lee la hora y el mes de tu teléfono y te dice qué puedes cazar en este momento. El filtro "Me faltan" deja solo lo que aún no has registrado. Arriba avisa de dos cosas que se pierden fácil: criaturas cuya franja horaria se cierra en menos de dos horas, y criaturas que están en su último mes del año. Con "Otro momento" puedes mirar cualquier mes y hora para planificar.

**Catálogo** — Las 200 criaturas con buscador. Funciona con o sin tildes y también con el nombre en inglés (`tiburon`, `Tiburón` y `great white shark` llevan al mismo pez). Se puede filtrar por tipo, por lo que te falta, por lo que ya tienes y por lo que está disponible ahora mismo.

**Ficha** — Al tocar cualquier criatura: precio en la tienda de los Nook, precio con C.J. o Flick, dónde aparece, tamaño de la sombra, velocidad, rareza, y dos bandas que muestran de un vistazo los 12 meses y las 24 horas en que aparece, con una marca en el mes y la hora actuales.

**Museo** — Porcentaje total y por categoría, más la lista de lo que te falta este mes, ordenada poniendo primero lo que se va a final de mes.

**Perfiles** — Varios personajes, cada uno con su hemisferio y su propio registro. El hemisferio cambia todos los meses de la app.

## Detalles que conviene saber

- **El progreso se guarda solo en el navegador de ese dispositivo** (`localStorage`). No se sincroniza entre teléfono y computador. Por eso el Museo tiene un botón para descargar tu registro como archivo `.json` y otro para restaurarlo. Si borras los datos del navegador, se borra el registro.
- **Funciona sin conexión** una vez que la has abierto por primera vez, gracias al *service worker*.
- **El salmón japonés y el salvelino** cambian de horario según el mes (de día en otoño, de tarde-noche en primavera). La app muestra la unión de sus dos ventanas, así que en esos dos peces la banda de horas es más generosa que el juego.
- **Las imágenes no vienen incluidas**, pero la app las soporta. Ver la sección de abajo.
- **La paleta** son los colores del juego: arena, verde hoja, cielo y ciruela. El fondo cambia de tinte con la hora del día (durazno al amanecer, cielo de día, rosa al atardecer, lavanda de noche).

## Añadir los dibujos de las criaturas (opcional)

La app busca una carpeta `iconos/` al arrancar. Si la encuentra, cada criatura aparece con su dibujo en la lista y en la ficha; si no está, se ve exactamente igual que ahora, sin huecos ni errores.

La estructura que espera es:

```
iconos/
  fish/   carp.png, koi.png, coelacanth.png…   (80)
  bugs/   tarantula.png, common_butterfly.png… (80)
  sea/    seaweed.png, vampire_squid.png…      (40)
```

Los nombres de archivo ya están dentro de `index.html`, en el campo `f` de cada criatura. Un juego de iconos de 128×128 con exactamente esos nombres está en la carpeta `icons/` del [repositorio ACNHAPI](https://github.com/alexislours/ACNHAPI):

```bash
git clone --depth 1 --filter=blob:none --sparse https://github.com/alexislours/ACNHAPI.git tmp
cd tmp && git sparse-checkout set icons && cd ..
mkdir -p iconos
cp -r tmp/icons/fish tmp/icons/bugs tmp/icons/sea iconos/
rm -rf tmp
```

Son 200 archivos, unos 2,2 MB. Se cachean solos para uso sin conexión la primera vez que los ves.

**Ten en cuenta que esas imágenes son recursos del juego y pertenecen a Nintendo.** Al subirlas a un repositorio público las estás redistribuyendo. Es lo que hacen casi todos los proyectos de fans de *New Horizons* y Nintendo no suele perseguir herramientas gratuitas y sin ánimo de lucro, pero no están licenciadas para reutilizarse: la decisión es tuya. Si prefieres evitarlo, la app funciona perfectamente sin la carpeta.

## Publicarlo en GitHub Pages

1. Crea un repositorio nuevo y sube estos archivos a la raíz (no dentro de una carpeta).
2. En el repositorio: **Settings → Pages**.
3. En *Source* elige **Deploy from a branch**, rama `main`, carpeta `/ (root)`. Guarda.
4. Al cabo de un minuto queda en `https://TU-USUARIO.github.io/NOMBRE-DEL-REPO/`.

Las rutas son todas relativas, así que funciona igual en la raíz del dominio o dentro de un subdirectorio.

## Instalarla en el teléfono

Ábrela en el navegador y añádela a la pantalla de inicio:

- **Android / Chrome**: menú ⋮ → *Añadir a pantalla principal*.
- **iPhone / Safari**: botón compartir → *Añadir a pantalla de inicio*.

Se abre a pantalla completa, sin barra de navegador, y arranca aunque no haya señal.

## Archivos

| Archivo | Para qué |
|---|---|
| `index.html` | **La app entera**: estilos, datos y lógica en un solo archivo |
| `sw.js` | Caché para uso sin conexión |
| `manifest.webmanifest` | Datos de instalación (nombre, iconos, colores) |
| `icono-192.png`, `icono-512.png` | Icono de la app |

Todo va en `index.html` a propósito: así se puede abrir con doble clic desde el disco, sin servidor. Dentro está ordenado en tres bloques con comentarios — estilos, luego `const DATOS`, y al final la lógica.

## Si editas algo

Cambia el número de versión en `sw.js`:

```js
const CACHE = 'isla-v1';   // → 'isla-v2', 'isla-v3'…
```

Sin eso, los teléfonos que ya abrieron la app seguirán mostrando la versión vieja desde la caché.

### Cómo están codificados los datos

Para que el bloque `DATOS` pese poco, los meses y las horas van como máscaras de bits:

- `N` y `S`: meses en el hemisferio norte y sur. El bit 0 es enero, el bit 11 es diciembre.
- `h`: horas del día. El bit 0 son las 00:00, el bit 23 son las 23:00. `16777215` = todo el día.
- `p` precio en la tienda, `c` precio con C.J. o Flick, `l` lugar, `s` sombra, `v` velocidad, `r` rareza.

Para comprobar si algo aparece en un mes: `(N >> (mes - 1)) & 1`.

## De dónde salen los datos

Los meses, horas, precios, sombras y lugares vienen del [ACNHAPI](https://github.com/alexislours/ACNHAPI), que a su vez recoge la [hoja de datos comunitaria de *New Horizons*](https://docs.google.com/spreadsheets/d/13d_LAJPlxMa_DubPTuirkIV4DERBMXbrWQsmSh8ReK4), la misma que usa [Nookipedia](https://nookipedia.com/wiki/Fish/New_Horizons). Los nombres en español son los oficiales del juego.

Proyecto de fan, sin relación con Nintendo. *Animal Crossing* es marca registrada de Nintendo.
