---
name: seguimiento-favoritas
description: Repasa las subastas que el usuario sigue en sus favoritas de AlertaSubastas, ordenadas por fecha de cierre, con el estado de seguimiento y los comentarios, y las actualiza cuando el usuario lo pide. Úsala cuando el usuario pregunte por sus favoritas, qué subastas acaban pronto, cómo van las que sigue, o quiera guardar, comentar, cambiar de estado o quitar una subasta de su lista.
---

# Seguimiento de favoritas

Las favoritas son la lista de trabajo del usuario en AlertaSubastas: cada una tiene un estado de seguimiento (`nueva`, `en_revision`, `interesante`, `preparando_puja`, `pujando`, `adjudicada`, `finalizada`, `descartada`) y un comentario libre. Necesitan una cuenta conectada con plan Profesional o Empresa.

## 1. Repaso

1. `listar_favoritas` (no consume cuota). Por defecto, las activas y las próximas: dos llamadas, con `subasta: activas` y con `subasta: proximas`. Las finalizadas (`subasta: finalizadas`) solo si el usuario las pide. Pagina con `limit` y `offset` si `total` es mayor que lo recibido.
2. Ordena por fecha de fin (`subasta`, fecha de fin), las más cercanas primero.
3. Presenta una tabla:

| Bien | Fin | Estado de la subasta | Mi estado | Comentario | Enlace |

- Marca las que acaban en los próximos 7 días.
- Señala incoherencias que el usuario querrá corregir: una favorita en `pujando` o `preparando_puja` cuya subasta ya ha finalizado, una en `nueva` que acaba en pocos días, una `descartada` que sigue activa.

El título de una activa es genérico (tipo, municipio, m², fin) hasta que se consulta su ficha. Si el usuario quiere el detalle, `ver_bien` lo da; si `consultada` es true, no gasta cuota, y si es false consume 1 de la cuota mensual: dilo antes de abrir varias.

## 2. Cambios

Cada cambio es una escritura en la cuenta del usuario: hazlo solo cuando lo pida, y confirma qué vas a cambiar si la petición es ambigua ("descarta las de Madrid" → enseña cuáles son antes).

- Guardar una subasta: `anadir_favorita` con el id o la URL y, si el usuario lo da, un comentario. Cuenta en el límite mensual de altas del plan.
- Comentar: `comentar_favorita`. El comentario se sustituye entero, así que si el usuario quiere añadir algo, parte del comentario actual.
- Cambiar de estado: `estado_favorita` con uno de los estados de arriba.
- Quitar: `quitar_favorita`. No devuelve el hueco del límite mensual: si el usuario solo quiere sacarla de su vista, propón el estado `descartada`.

## 3. Cuando una favorita finaliza

Si una favorita ha finalizado, `ver_bien` da el estado final y la puja con la que acabó, si se publicó. Ofrece pasarla a `finalizada` (o `adjudicada` si el usuario dice que la ganó) y guardar el resultado en el comentario.
