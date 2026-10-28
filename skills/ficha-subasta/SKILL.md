---
name: ficha-subasta
description: Reúne en una ficha clara todo lo publicado sobre un bien en subasta pública en España (bien, lote y subasta) con el conector de AlertaSubastas, y señala qué hay que comprobar antes de pujar. Úsala cuando el usuario comparta una URL o un id de AlertaSubastas, una referencia catastral, un IDUFIR, un id del BOE (SUB-…) o una matrícula, o pida "la ficha", "los datos", "qué sabemos" o "qué mirar" de una subasta concreta.
---

# Ficha de una subasta

Una ficha resume lo que dicen los datos publicados de un bien y lista lo que falta comprobar. No es un análisis jurídico: no decide si una carga subsiste ni si conviene pujar.

## 1. Encontrar el bien

- URL o id de AlertaSubastas (`0x…`): directo a `ver_bien`.
- Referencia catastral, IDUFIR/CRU, referencia registral, id del BOE (`SUB-…`), id de la Seguridad Social o la AEAT, anuncio del BOE, bastidor o matrícula: `buscar_por_codigo`, que no consume cuota. Si devuelve varios ids, enséñalos y pregunta cuál.

`ver_bien` consume 1 de la cuota mensual la primera vez que se consulta cada bien; repetirla es gratis.

## 2. Completar el lote

Si `lote.TOTAL_BIENES` es mayor que 1, los bienes se adjudican juntos: `ver_lote` da los ids de los demás (no consume cuota). Dilo en la ficha y pregunta antes de abrir sus fichas, porque cada una consume cuota.

## 3. Ficha

Usa estos apartados y solo los datos que devuelve `ver_bien`. Si un campo viene vacío, `null` o `desconocido`, escribe "no publicado"; no lo deduzcas.

**Qué es**
- `bien.TITULO`, `TIPO_BIEN`, dirección (`CALLE`, `NUMERO`, `RESTO_NUMERO`, `MUNICIPIO`, `PROVINCIA`), m² del `resumen`.
- `REF_CATASTRAL`, `IDUFIR`, `REF_REGISTRAL` y la descripción registral (`DESCRIPCION`).
- `VIVIENDA_HABITUAL`: si es la vivienda habitual del deudor. Cambia las reglas de adjudicación en las subastas judiciales.

**La subasta**
- Origen y procedimiento: `subasta.CLASE_SUBASTA.nombre` y `PROCEDIMIENTO.nombre`.
- Identificadores: `IDSUBASTA` y `ID_ANUNCIO`, con `subasta.URL` (página oficial).
- Estado y fechas: `ESTADO.nombre`, fin en `lote.ESTADO_FECHA` o `subasta.FECHA_FIN_CELEB`, publicación en `FECHA_PUBLICACION`.

**Importes**
- Valor de subasta (`bien.VALOR`), tasación (`lote.TASACION` o `subasta.TASACION`, si es distinta de 0), depósito (`DEPOSITO`), cantidad reclamada (`lote.RECLAMADA` o `subasta.RECLAMADA`), puja mínima (`lote.PRECIO_MINIMO`, si es mayor que 0), tramos (`TRAMOS`) y puja actual (`PUJA`).
- Valor de mercado: `valor_mercado.venta` y `alquiler_mes`, de Idealista. Si `aproximado` es true, es el precio medio de la zona por la superficie, no una valoración del inmueble. Indica la diferencia con el valor de subasta en euros y en %, sin interpretarla como ganancia.

**Situación jurídica declarada**
- Cargas: `CARGAS.estado`, `CARGAS.importe` y `CARGAS_DESC`, tal como las publica la subasta. No digas si subsisten o se cancelan: eso depende de la nota simple y del orden de las inscripciones.
- Posesión: `SITUACION_POSESORIA.estado` y `detalle`.
- Visitas: `VISITABLE`.
- `TITULO_JURIDICO`, `INSCRIPCION` e `INFO_ADICIONAL`, si los hay.

**Similares**
- `similares` trae ids de subastas cercanas con su distancia. Ofrece abrirlas si el usuario quiere comparar.

Cierra con el enlace `url` a la ficha en AlertaSubastas.

## 4. Qué comprobar antes de pujar

Lista solo los puntos que apliquen a este bien, según lo que falte o lo que diga la ficha:

- **Nota simple actualizada** del Registro de la Propiedad: cargas, titularidad y orden de inscripciones. Hazlo siempre que las cargas sean `desconocido` o haya cargas declaradas.
- **Ocupación**: si la situación posesoria es desconocida o hay ocupantes, cómo se recupera la posesión en este tipo de subasta.
- **Deudas que pasan al comprador**: comunidad de propietarios e IBI pendientes.
- **Visita**: si `VISITABLE` no lo indica, preguntar al órgano que lleva la subasta.
- **Lote con varios bienes**: se puja por todos juntos.
- **Reglas de esta clase de subasta**: depósito, cuándo se adjudica, plazos para pagar el resto, cesión de remate. Cambian entre judiciales, notariales, Seguridad Social y AEAT.

Para explicar cualquiera de estos conceptos usa `explicar_concepto` con la pregunta concreta (por ejemplo "cuándo se adjudica una subasta judicial de vivienda habitual") y cita la URL del artículo que devuelve. No des porcentajes, plazos ni reglas legales de memoria.

No digas si conviene pujar ni cuánto. Si el usuario lo pide, explica qué datos tiene que reunir para decidirlo y que la decisión es suya o de un profesional.
