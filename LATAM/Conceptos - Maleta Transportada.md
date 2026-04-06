# Sistemas
## Digital Workspace
Plataforma interna corporativo que permite el conectado a SABRE usando en el gate, mayormente en los aeropuertos de Brasil.
## BRS (Controla el equipaje)
Recibe de SABRE los BSM (no necesariamente un equipaje facturado)
Administra los BPM (carga y descarga) para la conciliación de equipaje.
## WorldTracer
- Gestión de irregularidades: Extravíos, equipaje sobrante, daños
- Se gestiona AHL, OHD, entre otros.
## SITA
Empresa que provee la red mundial de telecomunicaciones en aerolineas y aeropuertos permitiendo la conexión entre SABRE, WTR, BRS. Enviando mensajes como BSM, BPM, AHL, FWD.
## RMB (Report My Bag)
Registro del equipaje extraviado usando un QR brindado por el agente. Una vez creado el AHL se le brinda un PIR.
- Pasajero escanea el QR del agente -> Ingresa PNR -> Completa datos -> Sistema crea un PIR en WTR
## SABRE
- Sistema que permite la creación del vuelo.
- Sistema donde se realiza el check-in.
	- Se realiza el check-in  (pax confirma que viajara) el cual consta:
		- Se registra el equipaje: Esto es diferente a comprar un equipaje, ya que esto se puede realizar antes del check-in en las diferentes plataformas.
		- Se imprime (crea) el bagtag (etiqueta)
		- Se genera el BSM
		- BRS recibe el BSM
	- El check in se realiza ya sea: online, app movil, kiosko, counter, gate (excepciones)
- Cuando el sistema se encuentra offline, normalmente se genera una etiqueta manual
- Un **equipaje comprado != equipaje facturado**
## BSM (Bagggage Source System)
Mensaje electrónico generado por el check-in del bagtag.
Contiene información como el bagtag, pnr, vuelo, ruta, nombre pax, otros.
Tiene como destino el BRS en su preferencia.
## BPM (Baggage Process Manage)
Mensaje que proviene de BRS indicando el movimiento físico del equipaje:
- Carga (rampa)
- Descarga (rampa)
- Bagroom
# Facturación maleta (lugares)
Se considera **equipaje facturado** si:
- Existe registro digital (BSM), es decir, se debe realizar un check-in
- Aceptación fisica por parte del aeropuerto
## Counter Tradicional
1. Agente realiza check in de la maleta en SABRE
2. Imprime Bagtag y se adhiere a la maleta
3. Genera BSM
4. Agente recibe fisicamente la maleta y lo envía a la cinta
## Kiosko + Self Bag Drop (SBD)
1. Pasajero hace check in en Kiosko (BSM)
2. Imprime la etiqueta
3. Pasajero deposita en SBD (Self Bag Drop) la maleta
4. Sistema confirma aceptación física
## Gate
Esto ocurre en vuelos llenos:
	1. Se imprime etiqueta en gate
	2. Se genera BSM
	3. Personal de rampa recibe la maleta
La etiqueta también puede ser manual pero esto genera riesgos, ya que mayormente suele ser más para operación y no se registra en el sistema.
# Tipos de Etiqueta
- **Etiqueta automatizada**
	- Generado por el sistema SABRE / BRS (puede ser rastreable)
- **Etiqueta manual**
	- Creado cuando hay sistema caído, no hay impresora, operación de contingencia
	- Solo ingresa a BSM cuando la etiqueta es ingresada a SABRE. 
	- Puede crear problemas como:
		- Generar AHL sin registro en BRS
		- Puede transformarse en búsqueda por excepción si no se regula
- **Etiqueta Offline**
	Son bagtag impresos desde SABRE que no están ligados a un pasajero y solo indican el aeropuerto de destino. Posteriormente alguien tiene que **activarlos** (por ejemplo usando **SBD** o llevandolo al **counter**) para que aparezca un BSM change, ya que al ser solo creados solo aparece el evento de NEW (INSERT).
	Sucede en el gate.
# Flujos de pérdida de maleta
## Caso normal
- El equipaje se factura , creación bagtag, se pierde (opcional) -> Se abre AHL
## Caso excepcional (AHL sin bagtag)
Busqueda por excepción
- Pasajeros sin comprobante de equipaje
- Equipaje no registrado
- Pasajeros con boletos separados
- Equipaje retenido por comprobante no permitido
- Caso de seguridad obligatoria
Estos casos no generan indeminización automática
## Caso RMB
- Se crea PIR
- No se crea etiqueta nueva
- Si el equipaje tiene etiqueta, mantiene su etiqueta
- Si no estaba con etiqueta, sigue sin etiqueta.

Compensaciones
- Se dan compensaciones a todas las maletas que tienen un bagtag o incluso a las que no tienen
- Todas las maletas deberían tener un bagtag ?, en que momento se asigna un bagtag ?
# Términos
- Bagroom belts: 
	Cintas transportadoras internas del bag room por donde pasan las maletas para realizar claisificacion, controles y envio hacia el make up area (donde se arma la carga por vuelo)
- Kiosko
	Maquina de autoservicio conectada a SABRE que permite:
	- Hacer check in
	- Imprimir tarjeta de embarque
	- Imprimir bagtag
	Debes de ir a SBD para poder entregar la maleta y que se aceptada (generación del BSM).
	Si el pasajero imprime la etiqueta pero no lo entrega en SBD,  se genera un bagtag y BSM de SABRE pero el equipaje se queda como expected, por lo tanto no se genera un BPM, provocando que se bloquee el embarco.





