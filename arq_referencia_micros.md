
### Índex de l'Arquitectura de referència de microserveis

- **[2.1 Criteris d’aplicabilitat](#criteris-aplicabilitat)**

- **[2.2 Principis de l’arquitectura de referència](#principis-arquitectura-referencia)**

- **[2.3 Models d’ús de l’arquitectura](#models-d-us-arquitectura)**
  * 2.3.1 Model de disseny funcional – distribució en microserveis
     + 2.3.1.1	Tècniques per definir perímetres segons el domini de negoci
	 + 2.3.1.2	Validació de la descomposició funcional
  * 2.3.2 Model de desenvolupament

- **[2.4 Arquitectura tècnica](#arquitectura-tecnica)**
  * 2.4.1 Disseny conceptual
     + 2.4.1.1 Arquitectura d’execució
	 + 2.4.1.2 Arquitectura d’operació
	 + 2.4.1.3 Arquitectura de desenvolupament
  * 2.4.2 Disseny lògic
     + 2.4.2.1 Disseny lògic neutre respecte a proveïdor cloud
	 + 2.4.2.2 Disseny lògic basat en el cloud públic Azure
  * 2.4.3 Disseny físic


# 2. Arquitectura de referència de microserveis

## <a name="criteris-aplicabilitat"></a>2.1 Criteris d’aplicabilitat 

L’ús de l’arquitectura aporta avantatges, especialment, relacionats amb la capacitat de lliurar canvis i evolucions de la solució més ràpidament als usuaris però també introdueix dificultats i complexitats que fan que la decisió sobre l’ús d’aquest tipus d’arquitectura sigui rellevant.

A banda de l’avantatge principal de la rapidesa en quant a canvis podem identificar-ne, a nivell d’exemple, d’altres:

* Permet l’ús de les tecnologies més adequades per les funcionalitats de l’aplicació tant a nivell de llenguatges de programació com de plataformes de base.
* Facilita l’evolució tecnològica de l’aplicació mitigant, d’aquesta forma, el risc d’obsolescència tecnològica.
* Escala més eficientment ja que permet una distribució més adequada de recursos

En quant a les dificultats que es deriven poden enumerar les següents:
* Més complexitat en els processos de DevOps pel fet de l’existència de més components desplegats
* El diagnòstic d’errors pot ser més complex degut a la intervenció de múltiples components per resoldre una interacció
* Riscos derivats de les arquitectures distribuïdes per ser més proclius a errades de les comunicacions subjacents.

És per aquest motiu que s’estableixen uns **criteris d’aplicabilitat** per ajudar a la presa de decisió sobre l’ús de l’arquitectura. Aquests criteris es divideixen en **motivacions** que poden fer que una arquitectura de microserveis pugui ser beneficiosa per a un sistema i **condicionants** que hauria de complir el sistema per a poder-la aplicar amb garanties d’èxit.
 
![Motivacions](/images/img2.png)

Per d’altre banda, l’ús de l’arquitectura també implica uns condicionants que es poden classificar en diferents aspectes: solució, procés de construcció, equip i infraestructura de desplegament. 

![Condicionants](/images/img3.png)
 

## <a name="principis-arquitectura-referencia"></a>2.2 Principis de l’arquitectura de referència 

L’objectiu principal de l’estil arquitectural descrit és **accelerar l’entrega de funcionalitats** de les solucions facilitant les pràctiques d’entrega i desplegament continuats. És important tenir en consideració que, tot i que l’estil d’arquitectura faciliti aquest punt, no és l’únic ingredient necessari. L’organització dels equips de desenvolupament i la metodologia utilitzada també tenen un pes molt important.
Des de la perspectiva de l’arquitectura, aquest objectiu s’aconsegueix, principalment, permetent que tot el **cicle de vida d’un component sigui el màxim independent possible de la resta**. Aquest punt s’alinea amb una organització on un equip sigui responsable, d’extrem a extrem, des dels requeriments fins l’operació, de cadascun d’aquests components.

Per aconseguir aquest objectiu és necessari establir uns **models d’ús de l’arquitectura** ja que disposar d’una adequada arquitectura tècnica no és, en cap cas, suficient per aconseguir-ho. Els models d’ús s’han estructurat amb model de disseny funcional i model DevOps. En el primer, es descriu l’aproximació al disseny funcional de la solució i, en el segon, com aquest disseny funcional es trasllada al desenvolupament i a l’operació.

Els dos models han estat guiats, prèviament, per uns principis que, de forma general, marquen quin ús s’ha de fer de l’arquitectura per treure’n el màxim profit de la mateixa. Els podem entendre com un visió més sintètica dels models desenvolupats a continuació

![Principis d'ús](/images/img4.png)
 

## <a name="models-d-us-arquitectura"></a>2.3 Models d’ús de l’arquitectura 

### 2.3.1 Model de disseny funcional – distribució en microserveis

L'objectiu del disseny funcional és la distribució adequada del conjunt de funcionalitats de la solució en diferents blocs funcionals. Una adequada distribució de les funcionalitats ha d'aconseguir que la major part de requeriments d'evolució de la solució impliquin la modificació d'un únic bloc funcional. 

Llavors, quina mida ha de tenir un bloc funcional? Podríem dir que ha de ser tant petit com es pugui i tant gran com els requeriments necessitin. 

D’una forma més concreta i per descriure millor aquest equilibri, un bloc funcional quant més petit és, més gestionable, ja que:

* Redueix el risc en els desplegaments donat que el canvi està molt acotat.
* Les proves de regressió són més àgils ja que el codi desplegat és menor.
* L'enteniment de la seva funcionalitat és més simple.

però, per d’altre banda, una descomposició en blocs funcionals excessivament petits pot provocar que l'evolució derivada d'un requeriment de negoci impacti a varis. Això implica múltiples desplegaments, possible coordinació entre equips de desenvolupament, etc. el que allunya de l'objectiu primer d'aquest tipus d'arquitectura.

Els principis bàsics que han de guiar la descomposició funcional són els ja identificats i desenvolupats fa dècades per Larry Constantine [YOU79] en els seus treballs de disseny estructurat: **alta cohesió** i **baix acoblament**. 

*Baix acoblament*

Quan un bloc funcional està poc acoblat amb un altre, un canvi en un d'ells no implica canviar l'altre. Què porta a un bloc funcional a estar més acoblat amb un altre? Per exemple;

* Estils d'integració dependents de tecnologia (RMI, etc)
* Necessitat de conèixer molta informació de l'altre bloc funcional
* Requerir de la invocació de moltes operacions diferents d’un altre bloc funcional

*Alta cohesió*

El codi que es modifica junt, que dona resposta a una funcionalitat concreta, ha d’estar arquitecturalment junt. Des d’aquesta perspectiva, poden haver diferents criteris per establir el perímetre dels blocs funcionals, per exemple:

* Domini de negoci: S’agrupen les funcionalitats definint els perímetres alineats amb les funcions identificades a nivell del negoci.
* Organitzacional: S’agrupen les funcionalitats alineant amb l’estructura dels equips de desenvolupament i les seves capacitats de treball conjunt i comunicació. Per exemple, en una solució on intervenen múltiples proveïdors podria tenir sentit modularitzar de forma que cadascun pugui ser autònom en la seva part.
* Dades: Es poden agrupar totes les funcionalitats que tenen relació amb un tipus de dada concreta, per exemple, dades bancàries candidates a haver de passar processos d’auditoria específics.
* Tecnologia: S’agrupen les funcionalitats segons la tecnologia que s’usa pel seu desenvolupament. Aquest enfocament pot tenir sentit en contextos on hi ha unes tecnologies clarament diferenciades per resoldre certes funcionalitats.

De forma general i salvant solucions amb necessitats molt específiques, el criteri de preferència és el domini de negoci i, en segona instància, l’organitzacional.

#### 2.3.1.1 Tècniques per definir perímetres segons el domini de negoci

La descomposició basada en aquest criteri requereix un alt coneixement funcional del domini i l’aplicació dels principis bàsics de disseny identificats en l’apartat anterior, és a dir, definir perímetres que tinguin un baix acoblament i una alta cohesió.

L’avantatge d’aquest criteri de descomposició és que existeixen tècniques o mètodes que poden facilitar la identificació adequada d’aquests perímetres sobresortint, entre aquestes, el **Domain-driven design** (DDD) [EVA04] i [VER13]. 

Domain Driven Design (DDD) és una aproximació al disseny de software que posa en el centre de l’activitat de disseny el domini de negoci. Els conceptes clau que se’n deriven serien:

* Domini: És el problema de negoci que s’està analitzant per a donar la solució.
* Ubiquitous Language: Es el llenguatge amb el qual els experts del negoci el descriuen i que ha de servir de nexe d’unió entre la descripció del domini i la codificació de la solució.
* Bounded context: És el concepte que proporciona DDD per acotar els diferents dominis.

La identificació de “bounded contexts” es correspon a la descomposició funcional que estem dissenyant. A banda de DDD, també s’han desenvolupat d’altres tècniques que ajuden a fer aquesta descomposició com, per exemple, **l’event storming** [BRA13] que és un exercici de brainstorming col·laboratiu que permet, també, delimitar aquests perímetres funcionals.

Per detall d’ambdós tècniques, consultar la bibliografia referenciada.

#### 2.3.1.2 Validació de la descomposició funcional

Tal com s’ha detallat a l’apartat, una adequada descomposició funcional és clau per poder aconseguir els objectius que persegueix l’arquitectura. Tot i que la millor forma d’assegurar que s’ha realitzat correctament és tenir un ampli coneixement del domini, aquí fem menció a alguns aspectes que poden ajudar a validar si la idoneïtat de la descomposició.

Per exemplificar ambdós aspectes ens basarem en una aplicació hipotètica Petstore que s’ha dividit en dos blocs funcionals:

* Botiga: Conté les funcionalitats que realitzar una comanda de compra sobre alguna mascota disponible així com el seguiment de les comandes per part del client i empleats de la botiga.
* Mascotes: Conté les funcionalitats de mostrar el catàleg de mascotes disponibles per vendre així com l’administració de les mateixes per part dels empleats de la botiga.


*"Chatty communications"*
En una arquitectura distribuïda les comunicacions entre components tenen un cost (rendiment, risc de fallida, etc) molt més elevat que en d’altres arquitectures. És per això que la necessitat d’una gran quantitat d’interaccions entre blocs funcionals per resoldre els escenaris d’ús principals d’una solució pot ser un indicador d’una descomposició que pot ser problemàtica.

La traducció dels escenaris d’us principals de la solució en diagrames de seqüència on es pugui veure les interaccions necessàries per resoldre’ls entre blocs funcionals es considera una bona eina per detectar problemes en el disseny.

A nivell d’exemple, es mostra el diagrama de seqüència de l’escenari d’alta de comanda a la PetStore on intervenen el bloc funcional de “Mascotes” i de “Botiga”. 

![Diagrama de seqüència](/images/img5.png)

Al diagrama es veuen clarament dos punts d’interacció entre ambdós blocs funcionals i que no hi ha un nombre gran d’interaccions entre ells per resoldre l’escenari.

*Models compartits*
La descomposició funcional de la solució també implica la descomposició dels models d’informació i la distribució del model global de la solució entre els diferents blocs funcionals propietaris. En aquesta descomposició del model d’informació tots els blocs funcionals poden requerir compartir parts del seu model amb d’altres. 

Si la quantitat d’informació a compartir implica que s’ha de replicar bona part del model entre blocs funcionals també és un indici d’una descomposició funcional problemàtica.

A nivell d’exemple, es mostra el diagrama global de dades de la PetStore i els diagrames dels blocs funcionals de “Mascotes” i de “Botiga”

![Diagrama global PetStore](/images/img6.png)

En els gràfics següents es pot veure com hi ha certes dades, concretament el nom de la mascota, que està en ambdós models però, en cap cas, hi ha cap rèplica de tot el model de dades. La “Botiga” únicament requereix un petit conjunt de dades de les mascotes i d’usuari per donar servei. 

![Diagrama Botiga](/images/img6a.png)

![Diagrama Mascotes](/images/img6b.png)  	 
	
### 2.3.2 Model de desenvolupament

#### 2.3.2.1	Patró principal de desenvolupament (BFF – Backend for frontend)

Per a cada bloc funcional, el desenvolupament hauria de seguir una metodologia BFF (Backend for frontend) adaptada a una arquitectura de microserveis.

Com a conseqüència, per a cada bloc funcional s'hauria de crear un únic projecte, que contindrà tant la part front com la part back. Això té l'avantatge que cada bloc funcional podria ser desenvolupat per equips diferents i independents. A més, cada bloc funcional hauria de disposar de la seva pròpia base de dades.

![Patró Backend for frontend](/images/img7.png)

* **Frontend**: Conjunt de micro-frontends que proporcionen les funcionalitats del bloc al usuari.
	* Només accedeix al seu backend.
	* Alt acoplament del front-end al backend
* **Backend**: Servei que implementa la lògica de negoci per donar resposta al seu frontend
	* Si cal, pot fer crides a serveis backend d'altres 
* **DB**: Base de dades pròpia per a cada bloc funcional

#### 2.3.2.2	Estructura de projecte

Per poder traçar el concepte de bloc funcional a través de totes les capes, en el repositori de codi font, els diferents components del projecte s’estructuraran dins d'un mateix projecte GIT organitzats en carpetes. S'hauria de crear una carpeta per al codi de la part front i una altra per la part back. A més, s'hauria de crear una carpeta per a la configuració que hauria de contenir les plantilles i les configuracions dels components d'infraestructura dels que està compost el bloc funcional. Això té com a conseqüència que les capes front i back del mateix bloc funcional tindran el mateix cicle de vida a nivell de desenvolupament, versionat i desplegament. Un exemple de l'organització d'un projecte GIT d'un bloc funcional podria ser la següent:

![Estructura de projecte](/images/img8.png)

| Avantatges      | Desavantatges |
| ----------- | ----------- |
| La gestió del codi i de la configuració es simplifica a causa de que tot està en el mateix projecte      | Un canvi de configuració requereix recompilar el projecte generant una nova versió de la imatge       |
| El codi i la configuració es versionen conjuntament   | |

Si algun requeriment no funcional requereix que en el bloc funcional hagi més d'un microservei o algun altre artefacte d'algun altre tipus es afegirà a la seva carpeta corresponent dins el projecte GIT del bloc funcional. De la mateixa manera, si per requeriments de projecte fos necessari que diferents components del mateix bloc funcional tinguin cicles de vida independents, es podria dividir el bloc funcional en diversos projectes GIT

#### 2.3.2.3	Descomposició funcional de l’estructura del projecte

En el cas en què un microservei d'un bloc funcional tingui requisits no funcionals particulars que diferencien o identifiquen dues o més parts diferenciades dins del microservei, es recomana dividir el microservei en diferents components (dins del mateix bloc funcional) per poder així aplicar la particularitat cadascun (p.e. escalat diferenciat). Aquesta descomposició es podria realitzar de dues formes diferents:

* Dins el mateix projecte GIT: Per a cada component identificat es podrien crear carpetes separades dins del mateix projecte GIT. Les plantilles de configuració de tots els components dels que es compon el projecte GIT estarien tots a la mateixa carpeta de configuració. Això té com a conseqüència que tots els components del projecte GIT tindrien el mateix cicle de vida a nivell de desenvolupament, versionat i desplegament.
* Projectes GIT separats: Es podria descompondre el bloc funcional en diversos projectes GIT. D'aquesta manera els components dels diferents projectes GIT tindrien cicles de vida separats i diferenciats a nivell de desenvolupament, versionat i desplegament.

Els requeriments de cada projecte determinaran l'estratègia a seguir per descompondre un bloc funcional en components

#### 2.3.2.4	Versionat

S'hauria de crear una única versió que afecta al mòdul funcional complet per igual, és a dir, el cicle de vida de les diferents capes del bloc funcional es gestiona amb una única versió de bloc funcional. A causa de que les plantilles de configuració formen part del mateix projecte, aquestes plantilles també es versionen conjuntament amb el codi de les capes front i back. Per tant, el codi de les dues capes i la configuració tindran el mateix cicle de vida a nivell de desenvolupament, versionat i desplegament.

![Versionat](/images/img9.png)

Si per requeriments del projecte s'ha hagut de dividir el bloc funcional en diversos projectes GIT, cada projecte s'hauria de versionar per separat, de manera que cada projecte tindrà un propi cicle de vida.

#### 2.3.2.5	Pipelines

Per a cada bloc funcional s'han de crear les següents pipelines:

* Una pipeline principal que construeix i desplega el bloc funcional complet: Aquesta pipeline hauria d'encarregar-se del desplegament de les dues capes, tant front com back. Primer s'hauria de construir els artefactes per a les capes front i back. A continuació, haurà de executar els tests automàtics i finalment hauria de fer el desplegament del bloc funcional complet als diferents entorns.
* Pipelines per desplegar versions concretes: Aquesta pipeline hauria de permetre el rollback a una versió concreta que ja s'havia desplegat anteriorment. Atès que el bloc funcional es versiona com a unitat, aquesta pipeline permetria fer rollback tant de codi com de configuració.
* Pipelines per realitzar rollbacks: Aquesta pipeline hauria de permetre el rollback a la versió anterior. 
* Si és possible, pipelines per aturar, arrencar i reiniciar els serveis del bloc funcional en els diferents entorns

#### 2.3.2.6	Metodologia de branching

Per a cada repositori GIT es recomana trunk-based development, adaptada a les necessitats de CTTI. El projecte disposarà únicament de la branca master i de branques feature on es realitzarà el desenvolupament de les diferents funcionalitats:
* **Master**:  És la branca principal de projecte, on va el codi per generar l'aplicació en entorns de producció.
* **Feature**: Branques generades a partir de la branca master. Sobre elles es faran els desenvolupaments de noves funcionalitats (o correccions de codi) i els canvis es tornen a passar a la branca master un cop finalitzats.

La pipeline de desplegament treballa exclusivament sobre la branca master i hauria de ser capaç de realitzar successivament els desplegaments als diferents entorns del CTTI. En el cas de disposar dels entorns d'Integració, Preproducció i Producció, la pipeline hauria de desplegar el bloc funcional primer a Integració, un cop superades satisfactòriament les fases de compilació i dels tests automàtics. Després d'haver validat el correcte funcionament de l'entorn d'Integració, es realitzaria el desplegament a Preproducció. A continuació, es realitzaran les proves UAT i un cop superades totes les validacions, el bloc funcional finalment es desplegaria a Producció. En la següent il·lustració s'esquematitza el funcionament de la pipeline en el cas de disposar només dels entorns de Preproducció i Producció. També es pot veure que l'execució de la pipeline es dispara després d'haver realitzat un merge d'alguna branca feature a la branca master.
 
![Branching](/images/img10.png) 

##### 2.3.2.7	Model de desplegament

A nivell de desplegament la capa back del bloc funcional es desplega en un contenidor dedicat i la capa front es desplega en un directori d'un contenidor front global, que serveix les capes front de tots els blocs funcionals de l'aplicació.

Avantatges:
* Ús més eficient dels recursos sense comprometre la independència dels equips.
* Es redueix la complexitat de les capes front
* El rendiment de les capes front millora, pel fet que tot es serveix des del mateix servidor HTTP
* El nombre de components desplegats és més reduït

A nivell de model de desplegament és important l'aïllament de recursos entre blocs funcionals

A la següent imatge es mostra el model de desplegament d'un bloc funcional amb: 
* Un microfrontend en Angular que es desplega en un Volum Persistent muntat al contenidor Nginx que serveix el frontal
* Un microservei en Java amb Spring Boot que es desplega en un POD independent

![Model de desplegament](/images/img11.png)

## <a name="arquitectura-tecnica"></a>2.4 Arquitectura tècnica 

### 2.4.1 Disseny conceptual

El disseny conceptual identifica i descriu els seus components des d’un punt de vista neutre tecnològicament i s’ha dividit en tres blocs:

* **Arquitectura d’execució**: Inclou els components necessaris per tal de poder executar els microserveis i microfrontends de la solució i que aquests interactuïn entre ells. El fet que l’arquitectura sigui altament distribuïda requereix l’existència de certs components que no són necessaris en arquitectures monolítiques. Aquests components són els que donen resposta als requeriments dels usuaris de negoci.

![Disseny conceptual. Arquitectura d’execució](/images/img15.png) 

* **Arquitectura d’operació**: Els components d’aquest bloc permeten que, un cop desplegada i en execució la solució, aquesta pugui ser supervisada i es pugui assegurar el seu nivell de servei. En arquitectures altament distribuïdes no és suficient una visió independent de cada artefacte de la solució sinó que calen visions agregades de monitorització, observabilitat, etc. Aquests components donen resposta als perfils d’operació de la solució.

![Disseny conceptual. Arquitectura d’operació](/images/img16.png)

* **Arquitectura de desenvolupament**: Inclou els components per poder desenvolupar i desplegar els microfrontends i microserveis que composen la solució. Per treure màxim profit de l’arquitectura és necessari que els diferents artefactes de la solució tinguin un cicle de vida el més autònom possible. Aquests components donen servei als equips de desenvolupament de la solució.

![Disseny conceptual. Arquitectura de desenvolupament](/images/img17.png)

#### 2.4.1.1 Arquitectura d’execució

La descripció dels blocs conceptuals de l’arquitectura d’execució de front-end són les següents:

![Disseny conceptual. Arquitectura d'execució - Frontend](/images/img18.png)

* **Aplicació contenidora**: És l’aplicació que té, com a responsabilitat principal, la composició dels diferents microfrontends creant un frontal únic i que farà de punt d’entrada de l’usuari de negoci.  No hauria de suposar una nova aplicació dins del sistema, sinó que es pot triar un dels blocs funcionals (normalment el que tingui una funcionalitat més core) perquè assumeixi la responsabilitat d'aplicació contenidora.
* **Microfrontends**: Components que descomponen l’aplicació en elements d’interfície d’usuari individuals i independents. Tenen un baix grau d'acoblament entre ells i poden tenir el seu propi cicle de vida de desplegament.  No es mostren a l'usuari per si mateixos, per a això, s'incrusten en l'aplicació contenidora que és la que compon les pantalles fent ús dels diferents microfrontends.
* **Serveis HTTP Client**: Serveis que controlen l’accés a les funcionalitats de backend i que proporcionen un punt d’accés comú a les mateixes. Homogeneïtzen l’accés al backend i poden afegir elements comuns a les peticions així com un tractament comú als errors de comunicació.
* **Serveis de sessió**: Serveis encarregats de mantenir i gestionar la informació d'interès relativa a la navegació de l'usuari que està usant l’aplicació.
* **Serveis d’integració**: Serveis que proporcionen mecanismes per implementar la integració entre els diferents microfrontends i l’aplicació contenidora. La integració entre microfrontends, en cas necessari, no és directa i usen aquests serveis.
* **Serveis d’autenticació i autorització**: Serveis que proporcionen les funcionalitats d’integració amb la gestió d’identitats i d’obtenció del token per accedir als serveis de backend. Addicionalment incorporen les funcionalitats d’autorització controlant qui pot fer què.

Els blocs conceptuals de l’arquitectura d’execució de backend són les següents:

![Disseny conceptual. Arquitectura d'execució - Backend](/images/img19.png)

* **API Gateway**: Proporciona un punt d’entrada únic a tots els microserveis de la solució aplicant les regles de redirecció internes i garantint-ne el control d’accés. És un component de redirecció lleuger orientat, principalment, a la publicació de les API’s pel front-end de la solució.
* **Serveis de descobriment**: Els serveis de descobriment implementen la lògica que interroga els serveis de registre per localitzar, en el moment de consum, la ubicació física del servei a invocar.
* **Serveis de registre**: Proporciona funcionalitats per tal que els serveis, en el moment de posar-se en marxa, hi publiquin la seva ubicació física associant-la a un nom i, d’aquesta forma, fer-se disponible pels serveis de descobriment.
* **Serveis d’integració**: Proporcionen mecanismes per implementar diferents estils d’integració entre els micro-serveis de la solució, per exemple, cues de missatgeria o events. D’igual forma proporcionen aquests serveis per integracions amb d’altres sistemes externs incloent, per determinades casuístiques, publicacions externes d’API’s.
* **Cache de memòria**: Emmagatzematge de dades d’alt rendiment orientat a eficientar l’accés als serveis de persistència. Addicionalment també pot emmagatzemar dades amb baixos requeriments de durabilitat com, per exemple, dades de sessió.
* **Microserveis**: Són els serveis que implementen la lògica pròpia de la solució i que són el resultat de la descomposició funcional de la mateixa. De forma general implementen també la lògica vinculada a d’altres requeriments no funcionals com, per exemple, monitorització, operació, etc.
* **Serveis de persistència**: Serveis de persistència de les dades d’alta durabilitat que proporcionen la possibilitat d’independitzar el magatzem de dades de cadascun dels microserveis.
* **Serveis d’autenticació i autorització**: Serveis que proporcionen el token necessari per accedir als serveis via l’API Gateway així com els rols associats a l’usuari autenticat.
* **Enrutament dinàmic i balanceig de càrrega**: Els components de balanceig de càrrega usen els serveis de descobriment per cercar on existeixen instàncies del servei sol·licitat i, d’aquesta forma, encaminar les peticions cap a una instància concreta. La funcionalitat garanteix detecció d’indisponiblitats per deixar d’enviar peticions. L’enrutament dinàmic permet decisions d’encaminament de peticions basat en regles de negoci.

#### 2.4.1.2 Arquitectura d’operació

La descripció dels blocs conceptuals de l’arquitectura d’operació són les següents:

![Disseny conceptual. Arquitectura d'operació](/images/img20.png)

* **Serveis de control**: Serveis que permeten realitzar accions de parada i arrancada dels serveis, auto-escalat, circuit breaker, healthiness- i readiness-checks, etc.
* **Serveis d’observabilitat**: Són els serveis que recopilen informació per donar una visió del comportament intern del sistema com, per exemple, per fer el seguiment de la traçabilitat de les peticions entre microserveis, etc.
* **Serveis de gestió de la configuració**: Proporciona capacitats de configuració a tots els microserveis de manera que no sigui necessària la distribució de fitxers de propietats. Els microserveis poden consumir la configuració al iniciar-se i actualitzen la configuració sense necessitat de reinicis.
* **Serveis de logging**: Serveis de centralització de la informació de logging generat pels diferents microserveis de forma que sigui explotable de forma global a nivell de solució i no de microservei concret.
* **Serveis de monitorització**: Serveis que permeten veure en temps real les mètriques que caracteritzen externament el comportament del sistema, per exemple, consums de memòria, CPU, temps de resposta, etc.

#### 2.4.1.3 Arquitectura de desenvolupament
La descripció dels blocs conceptuals de l’arquitectura de desenvolupament són les següents:

![Disseny conceptual. Arquitectura de desenvolupament](/images/img21.png)

* **Entorn de desenvolupament i eines de construcció**: És un programa o conjunt de programes que engloben totes les tasques necessàries per al desenvolupament d'un programa o aplicació. A més també inclou les eines de construcció i gestió de les dependències.
* **Repositori d’artefactes**: Emmagatzema i proporciona accés a paquets compilats de software amb suport multi-llenguatge.
* **Servei de control de versions**: Emmagatzema el codi suportant el versionat del mateix i proporcionat les funcionalitats de branching, merge, etc.
* **Proves de rendiment**: Eines d’automatització de les proves de rendiment per garantir que els temps de resposta són adequats a diferents nivells de càrrega.
* **Gestió de casos de prova**: Eines de gestió de proves per emmagatzemar informació sobre com s’han de fer les proves, planificacions de les activitats de prova i informar-ne sobre l’estat i resultats.
* **Prova funcional**: Eines d’automatització de les proves funcionals 
* **Garantia de qualitat de codi**: Eina d’inspecció de la qualitat del codi multi llenguatge que ofereix informes reportant aspectes com indicadors de la complexitat del codi, duplicitats, vulnerabilitats de seguretat, etc.
* **Serveis CI/CD**: Serveis que implementen el procés d’automatització de la construcció i desplegament en els diferents entorns incorporant cicles de proves cada vegada que un membre de l’equip de desenvolupament fa canvis al servei de control de versions.

De forma general, en aquest apartat s’entén el codi des d’un punt de vista ampli incloent les definicions d’infraestructura 

### 2.4.2 Disseny lògic

El disseny lògic és una realització concreta del disseny conceptual especificant les tecnologies que poden resoldre cada bloc. com s’ha comentat anteriorment poden haver diferents realitzacions però, en aquest cas, n’hem especificat dues a mode d’exemple:

* Disseny lògic neutre respecte a proveïdor de cloud
* Disseny lògic basat en el cloud públic Azure

En els dos dissenys lògics realitzats s'ha tingut en compte un escenari en el qual els microserveis corren en un únic CPD i sobre una única plataforma d'orquestració Kubernetes o OpenShift.

#### 2.4.2.1 Disseny lògic neutre respecte a proveïdor cloud

##### 2.4.2.1.1	Arquitectura d’execució

La descripció dels blocs lògics de l’arquitectura d’execució de frontend són les següents:

![Disseny lògic. Arquitectura d'execució](/images/img22.png)

* **Aplicació contenidora**
* **Microfrontends**
* **Serveis HTTP Client**
* **Serveis de sessió**
* **Serveis d’autenticació i autorització**
* **Serveis d’integració**

Els blocs lògics de l’arquitectura d’execució de backend són les següents:

* **API Gateway**
* **Serveis de descobriment, serveis de registre, enrutament dinàmic i balanceig de càrrega**: OpenShift o Kubernetes
* **Serveis d’integració**: Integracions síncrones (HTTP) amd API Manager o integracions asíncrones (publish-subscribe) amb Apache Kafka o RabbitMQ.
* **Cache de memòria**: Redis
* **Microserveis**: Conteneritzats sobre una plataforma d’orquestració (Openshift/Kubernetes) o serverless en un cloud públic o privat
* **Serveis de persistència**: Dependran fortament de la problemàtica concreta a resoldre per la Solució.
* **Serveis d’autenticació i autorització**: GICAR amb l'agent de Shibboleth del CTTI basat en Apache (https://git.intranet.gencat.cat/3048-intern/imatges-docker/gicar-shibboleth-openshift/tree/1.0.3) o Nginx (https://git.intranet.gencat.cat/3048-intern/imatges-docker/gicar-nginx-openshift/tree/1.0.0). 

##### 2.4.2.1.2	Arquitectura d’operació

La descripció dels blocs lògics de l’arquitectura d’operació són les següents:

![Disseny lògic. Arquitectura d'operació](/images/img26.png)

* **Serveis de control**: Kubernetes
* **Serveis d’observabilitat**: Jaeger, Kiali i Istio
* **Serveis de gestió de la configuració**: ConfigMaps i Secrets
* **Serveis de logging**: Stack EFK (Elasticsearch, Fluentd, Kibana)
* **Serveis de monitorització**: Prometheus i Grafana

##### 2.4.2.1.3	Arquitectura de desenvolupament

A continuació, es descriuen els blocs lògics de l'arquitectura de desenvolupament, basant-se majoritàriament en les eines de les que disposa l'SIC (Servei d’Integració Continua) del CTTI. De forma general, en aquest apartat s'entén el codi des de un punt de vista ampli incloent també les definicions d’infraestructura.

![Disseny lògic. Arquitectura de desenvolupament](/images/img27.png)

* **Entorn de desenvolupament i eines de construcció**: Per exemple, Eclipse (STS), Visual Studio Code, Webpack, Npm o Apache Maven
* **Repositori d’artefactes**: Nexus i Harbor
* **Servei de control de versions**: GitLab
* **Proves de rendiment**: Microfocus Load Runner
* **Gestió de casos de prova**: Microfocus ALM Octane
* **Proves funcionals**: Selenium i Cucumber 
* **Garantia de qualitat de codi**: SonarQube
* **Serveis CI/CD**: Jenkins

#### 2.4.2.2 Disseny lògic basat en el cloud públic Azure

##### 2.4.2.2.1	Arquitectura d’execució

La descripció dels blocs lògics de l’arquitectura d’execució de frontend és la mateixa que en el disseny neutre pel que fa a el proveïdor de cloud, per tant, en aquest punt només es descriuen els punts referits a serveis de backend.

![Disseny lògic Azure. Arquitectura d'execució](/images/img28.png)

* **Azure API Gateway**
* **Serveis de descobriment, serveis de registre, enrutament dinàmic i balanceig de càrrega**: Azure Kubernetes Service (AKS)
* **Serveis d’integració**: Integracions síncrones (HTTP) amb Azure API Manager o integracions asíncrones (publish-subscribe) amb Confluent Cloud.
* **Cache de memòria**: Azure Cache for Redis
* **Microserveis**: Com contenidors sobre AKS o amb Azure Functions (Serverless)
* **Serveis de persistència**: Les diferents tipologies de bases de dades, com per exemple Azure Database for PostgreSQL, MongoDB Atlas, Azure SQL Database, Instància administrada de Azure SQL, SQL Server a Virtual Machines, Azure Database for MySQL, Azure Database for MariaDB, Azure Cosmos DB, Azure Cache for Redis, Azure Database Migration Service, Azure Managed Instance for Apache Cassandra
* **Serveis d’autenticació i autorització**

##### 2.4.2.2.2	Arquitectura d’operació

La descripció dels blocs lògics de l’arquitectura d’operació són les següents:

![Disseny lògic Azure. Arquitectura d'operació](/images/img29.png)

* **Serveis de control**: Azure Kubernetes Service (AKS)
* **Serveis de gestió de la configuració**: Kubernetes ConfigMaps i secrets amb Azure Key Vault
* **Serveis de monitorització, de logging i d’observabilitat**: Azure Monitor i Azure App Insights

##### 2.4.2.2.3	Arquitectura de desenvolupament

De forma general, en aquest apartat s'utilitzaran les eines descrites amb detall en l'apartat: Disseny lògic neutre pel que fa al cloud / Arquitectura de desplegament on es fa referència a eines SIC i no SIC proporcionades per CTTI per resoldre tots els serveis necessaris en aquest building block.


### 2.4.3 Disseny físic
El disseny físic identifica i descriu els components que formen part o integren el sistema, i com aquests es configuren o interactuen. Aquest disseny físic és la implementació de el disseny lògic basat en cloud públic d’Azure.

![Disseny físic basat en Azure](/images/img30.png)

El disseny físic es basa en el disseny lògic basat en el cloud públic d'Azure. Dels serveis que ofereix Azure s'utilitzen sobre tot **Azure Kubernetes Service (AKS)**, **Grup de recurs**, **Application Gateway**, **Azure File Storage**, **Azure Cache**, **Azure Pipelines**, **Azure Repos**, **Azure Artifacts**, **Azure Container Registry**, **Azure Test Plans**, **Azure Key Vault**, **Azure Monitor** i **Azure App Insigths**. A més s'utilitzen **MongoDB Atlas** y **Confluent Cloud**.
