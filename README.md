# AlertaSubastas para Claude

Plugin de [AlertaSubastas](https://alertasubastas.com) para buscar y seguir subastas públicas de España desde Claude: judiciales y de la AEAT del Portal de Subastas del BOE, de la Seguridad Social, notariales y de ayuntamientos.

Incluye el conector MCP de AlertaSubastas y tres skills que enseñan a Claude a usarlo bien.

## Qué incluye

**Conector MCP** (`.mcp.json`): apunta a `https://alertasubastas.com/mcp`, el mismo servidor que el conector AlertaSubastas del directorio de Claude. Si ya tienes el conector, el plugin usa las mismas tools.

**Skills**

- `buscar-subastas`: convierte una petición ("pisos en subasta en Girona por debajo de 150.000 €") en filtros exactos, presenta los resultados en una tabla comparable y ofrece crear una alerta por email con los mismos filtros.
- `ficha-subasta`: reúne lo publicado sobre un bien (bien, lote, subasta, importes, cargas declaradas, posesión, valor de mercado) y lista qué comprobar antes de pujar. Encuentra el bien también por referencia catastral, IDUFIR, id del BOE o matrícula.
- `seguimiento-favoritas`: repasa tus favoritas por fecha de cierre, con su estado y tus comentarios, y las actualiza cuando lo pides.

## Cuenta y planes

Sin cuenta puedes buscar, ver cuántas subastas activas coinciden con tu búsqueda y consultar las subastas finalizadas, con límites diarios. Al conectar una cuenta de AlertaSubastas con plan Profesional o Empresa se abren todas las subastas activas, las alertas y las favoritas, con los cupos mensuales del plan.

La conexión usa OAuth: inicias sesión en AlertaSubastas y eliges si das permiso de solo lectura o de lectura y escritura. Puedes revocarlo cuando quieras desde Tu cuenta.

## Qué datos envía

El plugin no ejecuta código en tu equipo ni contiene scripts. Las únicas peticiones salen del conector MCP hacia `https://alertasubastas.com/mcp`, con los filtros y los identificadores que Claude usa en cada tool. Las skills son instrucciones en texto. El tratamiento de los datos se describe en la [política de privacidad](https://alertasubastas.com/info/privacidad) (apartado 8, conexión con asistentes de IA).

Las alertas, favoritas, comentarios y estados solo se crean o cambian cuando lo pides, y las skills piden confirmación antes de crear una alerta.

## Aviso

Los datos vienen de los boletines y portales oficiales de subastas y de estimaciones de mercado de terceros. Las skills describen los datos publicados; no son asesoramiento jurídico ni de inversión. Antes de pujar, consulta la nota simple del Registro y la documentación oficial de la subasta.

## Documentación y soporte

- Documentación del API y del MCP: https://alertasubastas.com/api/docs#mcp
- Soporte: soporte@alertasubastas.com

## Licencia

MIT. Ver [LICENSE](LICENSE).
