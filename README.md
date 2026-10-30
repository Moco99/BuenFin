# frame · Tienda del Buen Fin

Tienda de práctica del **AWS Student Builder Group ITVER**. La construyes en 6 sesiones, con servicios de AWS, hasta que aguante una carga tipo Buen Fin.

> **Los pagos son una simulación.** PayMate es una pasarela ficticia: no se cobra nada y no hay que escribir datos reales de tarjeta, nunca.

## Qué contiene este repositorio

Solo la parte web. El código de las Lambdas y el script de carga se entregan en la guía de cada sesión.

| Archivo | Qué es |
| --- | --- |
| `index.html` | La tienda. Es el mismo archivo para todas las sesiones: sus funciones se activan al llenar `CONFIG` (ver más abajo). |
| `pago.html` | PayMate, la pasarela simulada. Se usa desde la sesión 5. |
| `products.json` | El catálogo: nombre, precio, categoría, imagen y video de cada producto. Viene **sin videos**. |
| `products.con-videos.json` | El mismo catálogo con los videos activados. Es una plantilla: no se sube a S3. |
| `imgs/` y `videos/` | Carpetas vacías donde van tus fotos y videos. |

## Fotos y videos (no vienen en el repositorio)

Las fotos y los videos se reparten aparte: **[enlace al paquete de fotos y videos]**. Descárgalas y copia los archivos dentro de `imgs/` y `videos/`. Git los ignora, así que no se suben por accidente.

Estos son los nombres exactos que espera `products.json`:

| Producto | Foto | Video (opcional) |
| --- | --- | --- |
| Playera básica | `imgs/playera.png` | `videos/playera.mp4` |
| Chamarra de mezclilla | `imgs/chamarra.png` | `videos/chamarra.mp4` |
| Jeans clásicos | `imgs/jeans.png` | `videos/jeans.mp4` |
| Tenis urbanos | `imgs/tenis.png` | `videos/tenis.mp4` |

**Si ves una letra gris en lugar de la foto**, falta el archivo o su nombre no coincide.

**Para activar los videos**, copia la plantilla sobre el catálogo:

```bash
cp products.con-videos.json products.json
```

**Si usas tus propias fotos o videos:**
- Cambia la ruta en `products.json` si el nombre o la extensión no son los de arriba. Un PNG renombrado a `.svg` no funciona.
- Fotos en vertical 4:5 (por ejemplo 800×1000) y de menos de 200 KB. JPG o WebP pesan menos que PNG.
- Videos en **MP4 con H.264**, de 5 a 15 segundos, verticales y de pocos MB. Con ffmpeg se convierten así:

```bash
ffmpeg -i original.mp4 -vf "scale=720:900:force_original_aspect_ratio=increase,crop=720:900" -an -c:v libx264 -crf 28 -preset slow -pix_fmt yuv420p -movflags +faststart playera.mp4
```

- Usa solo material tuyo, generado por ti o con licencia que permita usarlo. Los derechos de lo que pongas son tuyos.

## Probarla en tu computadora

No abras `index.html` con doble clic: el navegador no deja leer `products.json` desde `file://`. Sírvela con un servidor local.

- **VS Code:** instala la extensión **Live Server**, abre la carpeta y pulsa **Go Live**.
- **Terminal:** `python3 -m http.server 8000` y abre `http://localhost:8000`.

Ábrela en **Chrome**. El navegador interno de VS Code no siempre reproduce los videos.

## Qué se sube a S3 en cada sesión

| Sesión | Qué subes | Después |
| --- | --- | --- |
| 1 · La vitrina | `index.html`, `products.json` y las carpetas `imgs/` y `videos/` | — |
| 2 · La puerta | `index.html` con tus valores de Cognito | invalidar `/index.html` |
| 3 · El mostrador | `index.html` con `apiUrl` | invalidar `/index.html` |
| 4 · La fila | nada | — |
| 5 · Los avisos | `pago.html` (nuevo) e `index.html` con `pasarelaUrl` | invalidar `/index.html` |
| 6 · El Buen Fin | nada | — |

Si cambias `products.json` o una imagen, también hay que invalidarlo en CloudFront (por ejemplo `/products.json` o `/imgs/*`), o seguirás viendo la versión anterior.

## Configuración (`CONFIG` en `index.html`)

Al inicio del `<script>` hay un bloque `CONFIG`. Viene vacío y lo llenas con lo que creas en cada sesión.

| Valor | Qué es | Sesión |
| --- | --- | --- |
| `cognitoDomain` | Dominio de Cognito, con `https://` y sin barra final | 2 |
| `clientId` | ID del cliente de la app en Cognito | 2 |
| `redirectUri` | Tu dirección de CloudFront, con barra final | 2 |
| `apiUrl` | La Invoke URL de tu API, sin barra final | 3 |
| `pasarelaUrl` | `'pago.html'` | 5 |

Mientras `clientId` esté vacío, la tienda funciona sin inicio de sesión. **Si publicas tu propia copia, no subas a GitHub tus valores de `CONFIG`.**

## Las 6 sesiones

1. **La vitrina:** el catálogo en un bucket de S3 cerrado y entregado por CloudFront con HTTPS.
2. **La puerta:** registro e inicio de sesión de clientes con Cognito.
3. **El mostrador:** el botón Pagar manda el pedido a una API de API Gateway protegida con Cognito.
4. **La fila:** los pedidos esperan en una cola SQS, con cola de errores, y una Lambda los guarda en DynamoDB.
5. **Los avisos:** cada pedido manda un correo por SNS, el cliente pasa a PayMate y una alarma de CloudWatch vigila la cola.
6. **El Buen Fin:** se simulan 200 pedidos, se ajusta la velocidad, se provoca una falla, se revisan los costos y se limpia la cuenta.

## Antes de empezar en AWS

- Usa **tu propia cuenta** y crea un presupuesto (**Budgets**) antes de construir nada.
- Todo se hace en la región `us-east-1`, y los recursos de la tienda llevan el prefijo `bf-`.
- **Al terminar, borra todo.** La sesión 6 trae el orden de limpieza.

## Licencia

El código se publica con licencia MIT (ver `LICENSE`). Las fotos y los videos no forman parte del repositorio ni de esa licencia.

Hecho por Arturo Báez Solano para el AWS Student Builder Group ITVER.
