---
name: buscar-subastas
description: Busca subastas públicas de España (BOE judiciales, notariales, Seguridad Social, AEAT, ayuntamientos…) con el conector de AlertaSubastas, convierte lo que pide el usuario en filtros exactos, presenta los resultados en una tabla comparable y ofrece crear una alerta con los mismos filtros. Úsala cuando el usuario pida buscar o listar subastas, pisos, casas, locales, garajes, solares o vehículos en subasta en una zona, por precio, superficie o fecha, o pregunte cuántas subastas hay de algo.
---

# Buscar subastas

Flujo para pasar de una petición en lenguaje natural ("pisos en subasta en Girona por debajo de 150.000 €") a una lista de subastas que el usuario pueda comparar, y de ahí a una alerta.

## 1. Traducir la petición a filtros

Los filtros de `buscar_bienes` solo aceptan valores exactos. No los adivines: sácalos de las tools, que no consumen cuota.

- **Zona**: `listar_provincias` da el slug de la provincia. Para un municipio o una comarca, `listar_municipios` con esa provincia da el nombre exacto del municipio y el slug de la comarca. `municipio` y `comarca` son excluyentes.
- **Tipo de bien, estados, clases de subasta, órdenes**: `opciones_filtros`. Para "inmuebles" en general usa el grupo `inmuebles`; para pisos, `vivienda`.
- **Precio**: `valoracion` es el valor de subasta en euros (`{min, max}`). Si el usuario habla de "lo que tengo que poner para pujar", es `deposito`.
- **Superficie**: `superficie` en m² construidos según catastro.
- **Estado**: por defecto `activas`. Usa `finalizadas` si pregunta por resultados o precios de adjudicación pasados.
- **Texto libre** (`q`): solo para lo que no cubre un filtro (una calle, "piscina", "con inquilino"…).

Si la zona es ambigua (un nombre que existe en varias provincias, "la costa", "cerca de Madrid"), pregunta antes de buscar.

## 2. Buscar

Llama a `buscar_bienes` con los filtros. Devuelve `total` y hasta 100 `ids` por página, sin datos. Cada búsqueda nueva consume 1 de la cuota mensual; repetirla en 60 minutos (también para pasar de página) es gratis.

- `total` = 0: di qué filtro es probablemente el que sobra y propón ampliar (zona vecina, comarca en vez de municipio, quitar el tope de precio).
- `total` muy alto (más de 100): propón acotar antes de abrir fichas.

## 3. Abrir las fichas justas

`ver_bien` da los datos de cada bien, pero la primera consulta de cada uno consume 1 de la cuota mensual de subastas consultadas. Los ids que vienen en `consultadas` ya están pagados.

- Abre por defecto las 5–10 primeras con el orden que más sentido tenga para la petición (`fecha_celebracion_asc` para "las que acaban pronto", `valoracion_asc` para "las más baratas").
- Antes de abrir muchas, mira cuánta cuota queda en `_limites` de la respuesta o con `mi_cuenta`, y pregunta si vale la pena gastar más.

## 4. Presentar

Tabla con una fila por bien:

| Bien | Municipio | Valor de subasta | Depósito | Puja actual | Fin | Enlace |

- **Bien**: `bien.TITULO` y, si es inmueble, los m² del `resumen`.
- **Valor de subasta**: `bien.VALOR`. Si hay `valor_mercado.venta.precio`, añade una columna con la estimación de mercado y di que es de Idealista (si `aproximado` es true, es la media de la zona, no una valoración del inmueble).
- **Puja actual**: `PUJA.importe` si existe; si no, el texto de `PUJA.estado` (`sin_pujas`, `secreta`…).
- **Fin**: `lote.ESTADO_FECHA` o `subasta.FECHA_FIN_CELEB`.
- **Enlace**: `url` (ficha en AlertaSubastas). La URL oficial del BOE u organismo está en `subasta.URL`.

Debajo, en dos o tres frases, lo que destaca (la que acaba antes, la que tiene más diferencia con el mercado, las que no tienen pujas). No recomiendes pujar ni des cifras de rentabilidad: describe los datos.

Usa solo los datos que devuelven las tools. Si un campo viene vacío o `desconocido`, dilo así.

## 5. Ofrecer la alerta

Al terminar, ofrece avisar por email de las subastas nuevas que cumplan esa búsqueda:

1. Llama a `alertas_previsualizar` con los mismos filtros (en las alertas `tipo_bien` es obligatorio; `texto_incluye` sustituye a `q`). Di cuántas subastas activas coinciden ahora.
2. Enseña al usuario los filtros y un título propuesto, y pide confirmación.
3. Solo con su sí, `crear_alerta`. Los filtros no se pueden editar después (para cambiarlos hay que borrar y crear otra), así que confirma bien.

Sin provincia la alerta es nacional, y los grupos de tipo de bien (`inmuebles`, `muebles`, `todos`) solo están en algunos planes: si `crear_alerta` responde con un error de plan, explícalo y propone una alerta por provincia o por tipo concreto.

## Sin cuenta conectada

Sin cuenta, `buscar_bienes` dice cuántas subastas activas coinciden (`total`), pero solo deja ver las finalizadas, con límites diarios. Las alertas y las favoritas necesitan una cuenta con plan Profesional o Empresa. En ese caso:

- Da el número de activas que coinciden.
- Ofrece ver las finalizadas parecidas como referencia de precios.
- Explica que para ver las activas y crear la alerta hay que conectar la cuenta de AlertaSubastas en el conector.
