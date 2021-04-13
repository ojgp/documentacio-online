### Índex de l'Arquitectura de referència de microserveis

- **[1 Criteris d’aplicabilitat](#criteris-aplicabilitat)**

- **[2 Principis de l’arquitectura de referència](#principis-arquitectura-referencia)**

- **[3 Models d’ús de l’arquitectura](#models-d-us-arquitectura)**
  * 3.1 Model de disseny funcional – distribució en microserveis
     + 3.1.1	Tècniques per definir perímetres segons el domini de negoci
	 + 3.1.2	Validació de la descomposició funcional
  * 3.2 Model de desenvolupament

- **[4 Arquitectura tècnica](#arquitectura-tecnica)**
  * 4.1 Disseny conceptual
     + 4.1.1 Arquitectura d’execució
	 + 4.1.2 Arquitectura d’operació
	 + 4.1.3 Arquitectura de desenvolupament
  * 4.2 Disseny lògic
     + 4.2.1 Disseny lògic neutre respecte a proveïdor cloud
	 + 4.2.2 Disseny lògic basat en el cloud públic Azure
  * 4.3 Disseny físic

- **[Referències](#referencies)**


# Arquitectura de referència de microserveis

## <a name="criteris-aplicabilitat"></a>1 Criteris d’aplicabilitat 

L’ús de l’arquitectura aporta avantatges, especialment, relacionats amb la capacitat de lliurar canvis i evolucions de la solució més ràpidament als usuaris però també introdueix dificultats i complexitats que fan que la decisió sobre l’ús d’aquest tipus d’arquitectura sigui rellevant.

És per aquest motiu que s’estableixen uns **criteris d’aplicabilitat** per ajudar a la presa de decisió sobre l’ús de l’arquitectura. Aquests criteris es divideixen en **motivacions** que poden fer que una arquitectura de microserveis pugui ser beneficiosa per a un sistema i **condicionants** que hauria de complir el sistema per a poder-la aplicar amb garanties d’èxit.

![Motivacions](/images/img2.png)

Per d’altre banda, l’ús de l’arquitectura també implica uns condicionants que es poden classificar en diferents aspectes: solució, procés de construcció, equip i infraestructura de desplegament.

![Condicionants](/images/img3.png)


## <a name="principis-arquitectura-referencia"></a>2 Principis de l’arquitectura de referència 

L’objectiu principal de l’estil arquitectural descrit és **accelerar l’entrega de funcionalitats** de les solucions facilitant les pràctiques d’entrega i desplegament continuats. És important tenir en consideració que, tot i que l’estil d’arquitectura faciliti aquest punt, no és l’únic ingredient necessari. L’organització dels equips de desenvolupament i la metodologia utilitzada també tenen un pes molt important.

Per aconseguir aquest objectiu és necessari establir uns **models d’ús de l’arquitectura** ja que disposar d’una adequada arquitectura tècnica no és, en cap cas, suficient per aconseguir-ho. Els models d’ús s’han estructurat amb model de disseny funcional i model DevOps. En el primer, es descriu l’aproximació al disseny funcional de la solució i, en el segon, com aquest disseny funcional es trasllada al desenvolupament i a l’operació.

Els dos models han estat guiats, prèviament, per uns principis que, de forma general, marquen quin ús s’ha de fer de l’arquitectura per treure’n el màxim profit de la mateixa. Els podem entendre com un visió més sintètica dels models desenvolupats a continuació

![Principis d'ús](/images/img4.png)
 

## <a name="models-d-us-arquitectura"></a>3 Models d’ús de l’arquitectura 

### 3.1 Model de disseny funcional – distribució en microserveis

L'objectiu del disseny funcional és la **distribució adequada del conjunt de funcionalitats de la solució en diferents blocs funcionals**. Una adequada distribució de les funcionalitats ha d'aconseguir que la major part de requeriments d'evolució de la solució impliquin la modificació d'un únic bloc funcional. 

Els principis bàsics que han de guiar la descomposició funcional són els ja identificats i desenvolupats fa dècades per Larry Constantine [YOU79] en els seus treballs de disseny estructurat: **alta cohesió** i **baix acoblament**. 

*Baix acoblament*
Quan un bloc funcional està poc acoblat amb un altre, un canvi en un d'ells no implica canviar l'altre. Què porta a un bloc funcional a estar més acoblat amb un altre? Per exemple;
* Estils d'integració dependents de tecnologia (RMI, etc)
* Necessitat de conèixer molta informació de l'altre bloc funcional
* Requerir de la invocació de moltes operacions diferents d’un altre bloc funcional

*Alta cohesió*
El codi que es modifica junt, que dona resposta a una funcionalitat concreta, ha d’estar arquitecturalment junt. 

Des d’aquesta perspectiva, poden haver diferents criteris per establir el perímetre dels blocs funcionals:

![Criteris per establir el perímetre dels blocs funcionals](/images/img4a.png)

### 3.2 Model de desenvolupament

#### 3.2.1	Patró principal de desenvolupament (BFF – Backend for frontend)

Per a cada bloc funcional, el desenvolupament hauria de seguir una metodologia BFF (Backend for frontend) adaptada a una arquitectura de microserveis.

Com a conseqüència, per a cada bloc funcional s'hauria de crear un únic projecte, que contindrà tant la part front com la part back. Això té l'avantatge que cada bloc funcional podria ser desenvolupat per equips diferents i independents. A més, cada bloc funcional hauria de disposar de la seva pròpia base de dades.

<img src="/images/img7.png" alt="Patró Backend for frontend" width="50%" />

* **Frontend**: Conjunt de micro-frontends que proporcionen les funcionalitats del bloc al usuari.
	* Només accedeix al seu backend.
	* Alt acoplament del front-end al backend
* **Backend**: Servei que implementa la lògica de negoci per donar resposta al seu frontend
	* Si cal, pot fer crides a serveis backend d'altres 
* **DB**: Base de dades pròpia per a cada bloc funcional

#### 3.2.2	Estructura de projecte

Per poder traçar el concepte de bloc funcional a través de totes les capes, en el repositori de codi font, els diferents components del projecte s’estructuraran dins d'un mateix projecte GIT organitzats en carpetes. S'hauria de crear una carpeta per al codi de la part front i una altra per la part back. A més, s'hauria de crear una carpeta per a la configuració que hauria de contenir les plantilles i les configuracions dels components d'infraestructura dels que està compost el bloc funcional. Això té com a conseqüència que les capes front i back del mateix bloc funcional tindran el mateix cicle de vida a nivell de desenvolupament, versionat i desplegament. Un exemple de l'organització d'un projecte GIT d'un bloc funcional podria ser la següent:

<img src="/images/img8.png" alt="Estructura de projecte" width="30%" />

| Avantatges      | Desavantatges |
| ----------- | ----------- |
| La gestió del codi i de la configuració es simplifica a causa de que tot està en el mateix projecte      | Un canvi de configuració requereix recompilar el projecte generant una nova versió de la imatge       |
| El codi i la configuració es versionen conjuntament   | |

Si algun requeriment no funcional requereix que en el bloc funcional hagi més d'un microservei o algun altre artefacte d'algun altre tipus es afegirà a la seva carpeta corresponent dins el projecte GIT del bloc funcional. De la mateixa manera, si per requeriments de projecte fos necessari que diferents components del mateix bloc funcional tinguin cicles de vida independents, es podria dividir el bloc funcional en diversos projectes GIT

#### 3.2.3	Descomposició funcional de l’estructura del projecte

En el cas en què un microservei d'un bloc funcional tingui requisits no funcionals particulars que diferencien o identifiquen dues o més parts diferenciades dins del microservei, es recomana dividir el microservei en diferents components (dins del mateix bloc funcional) per poder així aplicar la particularitat cadascun (p.e. escalat diferenciat). Aquesta descomposició es podria realitzar de dues formes diferents:

* Dins el mateix projecte GIT: Per a cada component identificat es podrien crear carpetes separades dins del mateix projecte GIT. Les plantilles de configuració de tots els components dels que es compon el projecte GIT estarien tots a la mateixa carpeta de configuració. Això té com a conseqüència que tots els components del projecte GIT tindrien el mateix cicle de vida a nivell de desenvolupament, versionat i desplegament.
* Projectes GIT separats: Es podria descompondre el bloc funcional en diversos projectes GIT. D'aquesta manera els components dels diferents projectes GIT tindrien cicles de vida separats i diferenciats a nivell de desenvolupament, versionat i desplegament.

Els requeriments de cada projecte determinaran l'estratègia a seguir per descompondre un bloc funcional en components

#### 3.2.4	Versionat

S'hauria de crear una única versió que afecta al mòdul funcional complet per igual, és a dir, el cicle de vida de les diferents capes del bloc funcional es gestiona amb una única versió de bloc funcional. A causa de que les plantilles de configuració formen part del mateix projecte, aquestes plantilles també es versionen conjuntament amb el codi de les capes front i back. Per tant, el codi de les dues capes i la configuració tindran el mateix cicle de vida a nivell de desenvolupament, versionat i desplegament.

![Versionat](/images/img9.png)

Si per requeriments del projecte s'ha hagut de dividir el bloc funcional en diversos projectes GIT, cada projecte s'hauria de versionar per separat, de manera que cada projecte tindrà un propi cicle de vida.

#### 3.2.5	Pipelines

Per a cada bloc funcional s'han de crear les següents pipelines:

* Una pipeline principal que construeix i desplega el bloc funcional complet: Aquesta pipeline hauria d'encarregar-se del desplegament de les dues capes, tant front com back. Primer s'hauria de construir els artefactes per a les capes front i back. A continuació, haurà de executar els tests automàtics i finalment hauria de fer el desplegament del bloc funcional complet als diferents entorns.
* Pipelines per desplegar versions concretes: Aquesta pipeline hauria de permetre el rollback a una versió concreta que ja s'havia desplegat anteriorment. Atès que el bloc funcional es versiona com a unitat, aquesta pipeline permetria fer rollback tant de codi com de configuració.
* Pipelines per realitzar rollbacks: Aquesta pipeline hauria de permetre el rollback a la versió anterior. 
* Si és possible, pipelines per aturar, arrencar i reiniciar els serveis del bloc funcional en els diferents entorns

#### 3.2.6	Metodologia de branching

Per a cada repositori GIT es recomana trunk-based development, adaptada a les necessitats de CTTI. El projecte disposarà únicament de la branca master i de branques feature on es realitzarà el desenvolupament de les diferents funcionalitats:
* **Master**:  És la branca principal de projecte, on va el codi per generar l'aplicació en entorns de producció.
* **Feature**: Branques generades a partir de la branca master. Sobre elles es faran els desenvolupaments de noves funcionalitats (o correccions de codi) i els canvis es tornen a passar a la branca master un cop finalitzats.

La pipeline de desplegament treballa exclusivament sobre la branca master i hauria de ser capaç de realitzar successivament els desplegaments als diferents entorns del CTTI. En el cas de disposar dels entorns d'Integració, Preproducció i Producció, la pipeline hauria de desplegar el bloc funcional primer a Integració, un cop superades satisfactòriament les fases de compilació i dels tests automàtics. Després d'haver validat el correcte funcionament de l'entorn d'Integració, es realitzaria el desplegament a Preproducció. A continuació, es realitzaran les proves UAT i un cop superades totes les validacions, el bloc funcional finalment es desplegaria a Producció. En la següent il·lustració s'esquematitza el funcionament de la pipeline en el cas de disposar només dels entorns de Preproducció i Producció. També es pot veure que l'execució de la pipeline es dispara després d'haver realitzat un merge d'alguna branca feature a la branca master.
 
![Branching](/images/img10.png) 

##### 3.2.7	Model de desplegament

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

<img src="/images/img8.png" alt="Model de desplegament" width="45%" />


## <a name="arquitectura-tecnica"></a>4 Arquitectura tècnica 

### 4.1 Disseny conceptual

El disseny conceptual identifica i descriu els seus components des d’un punt de vista neutre tecnològicament i s’ha dividit en tres blocs:

* **Arquitectura d’execució**: Inclou els components necessaris per tal de poder executar els microserveis i microfrontends de la solució i que aquests interactuïn entre ells. El fet que l’arquitectura sigui altament distribuïda requereix l’existència de certs components que no són necessaris en arquitectures monolítiques. Aquests components són els que donen resposta als requeriments dels usuaris de negoci.

![Disseny conceptual. Arquitectura d’execució](/images/img15.png) 

* **Arquitectura d’operació**: Els components d’aquest bloc permeten que, un cop desplegada i en execució la solució, aquesta pugui ser supervisada i es pugui assegurar el seu nivell de servei. En arquitectures altament distribuïdes no és suficient una visió independent de cada artefacte de la solució sinó que calen visions agregades de monitorització, observabilitat, etc. Aquests components donen resposta als perfils d’operació de la solució.

![Disseny conceptual. Arquitectura d’operació](/images/img16.png)

* **Arquitectura de desenvolupament**: Inclou els components per poder desenvolupar i desplegar els microfrontends i microserveis que composen la solució. Per treure màxim profit de l’arquitectura és necessari que els diferents artefactes de la solució tinguin un cicle de vida el més autònom possible. Aquests components donen servei als equips de desenvolupament de la solució.

![Disseny conceptual. Arquitectura de desenvolupament](/images/img17.png)

#### 4.1.1 Arquitectura d’execució

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

#### 4.1.2 Arquitectura d’operació

La descripció dels blocs conceptuals de l’arquitectura d’operació són les següents:

![Disseny conceptual. Arquitectura d'operació](/images/img20.png)

* **Serveis de control**: Serveis que permeten realitzar accions de parada i arrancada dels serveis, auto-escalat, circuit breaker, healthiness- i readiness-checks, etc.
* **Serveis d’observabilitat**: Són els serveis que recopilen informació per donar una visió del comportament intern del sistema com, per exemple, per fer el seguiment de la traçabilitat de les peticions entre microserveis, etc.
* **Serveis de gestió de la configuració**: Proporciona capacitats de configuració a tots els microserveis de manera que no sigui necessària la distribució de fitxers de propietats. Els microserveis poden consumir la configuració al iniciar-se i actualitzen la configuració sense necessitat de reinicis.
* **Serveis de logging**: Serveis de centralització de la informació de logging generat pels diferents microserveis de forma que sigui explotable de forma global a nivell de solució i no de microservei concret.
* **Serveis de monitorització**: Serveis que permeten veure en temps real les mètriques que caracteritzen externament el comportament del sistema, per exemple, consums de memòria, CPU, temps de resposta, etc.

#### 4.1.3 Arquitectura de desenvolupament
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

### 4.2 Disseny lògic

El disseny lògic és una realització concreta del disseny conceptual especificant les tecnologies que poden resoldre cada bloc. Per a cada bloc s'indiquen tecnologies candidates per als diferents serveis. Com s’ha comentat anteriorment poden haver diferents realitzacions però, en aquest cas, s'han especificat dues a mode d’exemple:

* Disseny lògic neutre respecte a proveïdor de cloud
* Disseny lògic basat en el cloud públic Azure

En els dos dissenys lògics realitzats s'ha tingut en compte un escenari en el qual els microserveis corren en un únic CPD i sobre una única plataforma d'orquestració Kubernetes o OpenShift.

#### 4.2.1 Disseny lògic neutre respecte a proveïdor cloud

##### 4.2.1.1 Arquitectura d’execució

La descripció dels blocs lògics de l’arquitectura d’execució són les següents:

![Disseny lògic. Arquitectura d'execució](/images/img22.png)

La capa frontend es compon dels següents blocs lògics :
* **Aplicació contenidora**: És l'aplicació que compon les distintes vistes que formen el frontal. Això s'aconsegueix incrustant els diferents microfrontends, a la aplicació contenidora, mitjançant l'ús dels Custom Elements.  Els Custom Elements incrustats encapsulen els diferents microfrontends. Al carregar les pàgines, l'aplicació contenidora, permet, al navegador, aixecar una nova instància de cada microfrontend incrustat. D'aquesta manera es renderitza el HTML de cada un i es disponibilitza la seva funcionalitat.
* **Microfrontends**: Cada bloc funcional ha d'implementar el seu propi microfrontend independent, de manera que s'asseguri la independència entre els diferents blocs funcionals. Els microfrontends podin ser desenvolupats en qualsevol llenguatge o framework de front-end acceptat per CTTI però és recomana mantenir l'homogeneïtat tecnològica dins de l’aplicació.
* **Serveis HTTP Client**: Els serveis HTTP Client centralitzen els accessos als serveis backend. Totes les crides que es realitzen des de la capa frontend als serveis backend es canalitzen a través d'aquest servei. D'aquesta manera, es pot utilitzar un servei HTTP Client per afegir de forma centralitzada propietats comunes a les crides a la capa backend, com ara la injecció transparent d'un token JWT per a l'autenticació correcta en els serveis backend. A més, es pot utilitzar un servei de HTTP Client per a la gestió centralitzada d'errors en les crides als serveis backend. Els serveis HTTP s'encarreguen d'interceptar tot els errors per mostrar a l'usuari un missatge d'error adequat.
* **Serveis de sessió**: Els serveis de sessió s'encarreguen de mantenir i gestionar la informació d'interès relativa a l'usuari que està utilitzant l'aplicació. Aquests serveis solen estar integrats amb els navegadors Web de cada usuari i solen guardar tant el token JWT de l’usuari como les seves propietats més rellevants, com el seu nom o els seus rols. Per això, els navegadors Web tenen incorporats un Session Storage, que es pot utilitzar per guardar dades temporals en el propi navegador de cada usuari. A més, també es poden utilitzar Cookies per guardar i gestionar informació relativa a la sessió de l'usuari en el seu navegador Web.
* **Serveis d’autenticació i autorització**: Els serveis d’autenticació proporcionen les funcionalitats d’integració amb la gestió d’identitats i d’obtenció del token per accedir als serveis de backend. Quan l'usuari encara no està autenticat, els serveis d'autenticació s'encarreguen de mostrar el formulari de login i gestionar el procés d'autenticació amb la capa back perquè aquesta generi un token JWT. En una aplicació per al CTTI l'autenticació d'usuari se sol delegar a GICAR, qui mostra els formularis de login i valida les credencials introduïts.
* **Serveis d’integració**: Els serveis d'integració centralitzen la comunicació entre els diferents components de la capa frontend. Per a això, l'aplicació contenidora es subscriu als esdeveniments emesos pels diferents microfrontends incrustats i els intercepta per propagar-les als microfrontends destí perquè aquests duguin a terme les accions corresponents, com ara un refresc de la interfície gràfica. De la mateixa manera, cada microfrontend hauria de disposar d'un servei que centralitza l'enviament de les seves diferents esdeveniments a l'aplicació contenidora.

Els blocs lògics de l’arquitectura d’execució de backend són les següents:

* **API Gateway**: Les funcionalitats de l'API Gateway es poden implementar amb un servei NGINX, utilitzant les imatges proporcionades pel CTTI que ja estan integrades amb Shibboleth.
* **Serveis de descobriment, serveis de registre, enrutament dinàmic i balanceig de càrrega**: OpenShift o Kubernetes
* **Serveis d’integració**: Integracions síncrones (HTTP) amd API Manager o integracions asíncrones (publish-subscribe) amb Apache Kafka o RabbitMQ.
* **Cache de memòria**: Com cache de memòria es pot utilitzar Redis
* **Microserveis**: Conteneritzats sobre una plataforma d’orquestració (Openshift/Kubernetes) o serverless en un cloud públic o privat
* **Serveis de persistència**: Dependran fortament de la problemàtica concreta a resoldre per la Solució.
* **Serveis d’autenticació i autorització**: GICAR amb l'agent de Shibboleth del CTTI basat en Apache (https://git.intranet.gencat.cat/3048-intern/imatges-docker/gicar-shibboleth-openshift/tree/1.0.3) o Nginx (https://git.intranet.gencat.cat/3048-intern/imatges-docker/gicar-nginx-openshift/tree/1.0.0). 

##### 4.2.1.2 Arquitectura d’operació

Les solucions proposades per als blocs lògics de l'arquitectura d’operació són les següents:

![Disseny lògic. Arquitectura d'operació](/images/img26.png)

* **Serveis de control**: Kubernetes
* **Serveis d’observabilitat**: Jaeger, Kiali i Istio
* **Serveis de gestió de la configuració**: ConfigMaps i Secrets
* **Serveis de logging**: Stack EFK (Elasticsearch, Fluentd, Kibana)
* **Serveis de monitorització**: Prometheus i Grafana

##### 4.2.1.3 Arquitectura de desenvolupament

A continuació, es mostren les solucions recomanades per als blocs lògics de l'arquitectura de desenvolupament, basant-se majoritàriament en les eines de les que disposa l'SIC (Servei d’Integració Continua) del CTTI. De forma general, en aquest apartat s'entén el codi des de un punt de vista ampli incloent també les definicions d’infraestructura.

![Disseny lògic. Arquitectura de desenvolupament](/images/img27.png)

* **Entorn de desenvolupament i eines de construcció**: Per exemple, Eclipse (STS), Visual Studio Code, Webpack, Npm o Apache Maven
* **Repositori d’artefactes**: Nexus i Harbor
* **Servei de control de versions**: GitLab
* **Proves de rendiment**: Microfocus Load Runner
* **Gestió de casos de prova**: Microfocus ALM Octane
* **Proves funcionals**: Selenium i Cucumber 
* **Garantia de qualitat de codi**: SonarQube
* **Serveis CI/CD**: Jenkins

#### 4.2.2 Disseny lògic basat en el cloud públic Azure

##### 4.2.2.1 Arquitectura d’execució

La descripció dels blocs lògics de l’arquitectura d’execució de frontend és la mateixa que en el disseny neutre pel que fa a el proveïdor de cloud, per tant, en aquest punt només es mostren els serveis d'Azure recomanades per a la capa backend.

![Disseny lògic Azure. Arquitectura d'execució](/images/img28.png)

* **API Gateway**: Com API Gateway es pot utilitzar Azure Gateway
* **Serveis de descobriment, serveis de registre, enrutament dinàmic i balanceig de càrrega**: Azure Kubernetes Service (AKS)
* **Serveis d’integració**: Integracions síncrones (HTTP) amb Azure API Manager o integracions asíncrones (publish-subscribe) amb Confluent Cloud.
* **Cache de memòria**: Com cache de memòria es pot utilitzar Azure Cache for Redis
* **Microserveis**: Com contenidors sobre AKS o amb Azure Functions (Serverless)
* **Serveis de persistència**: Les diferents tipologies de bases de dades, com per exemple Azure Database for PostgreSQL, MongoDB Atlas, Azure SQL Database, Instància administrada de Azure SQL, SQL Server a Virtual Machines, Azure Database for MySQL, Azure Database for MariaDB, Azure Cosmos DB, Azure Cache for Redis, Azure Database Migration Service, Azure Managed Instance for Apache Cassandra
* **Serveis d’autenticació i autorització**

##### 4.2.2.2 Arquitectura d’operació

Per l'arquitectura d'operació es podrien utilitzar els següents serveis d'Azure:

![Disseny lògic Azure. Arquitectura d'operació](/images/img29.png)

* **Serveis de control**: Azure Kubernetes Service (AKS)
* **Serveis de gestió de la configuració**: Kubernetes ConfigMaps i secrets amb Azure Key Vault
* **Serveis de monitorització, de logging i d’observabilitat**: Azure Monitor i Azure App Insights

##### 4.2.2.3 Arquitectura de desenvolupament

De forma general, en aquest apartat s'utilitzaran les eines descrites amb detall en l'apartat: Disseny lògic neutre pel que fa al cloud / Arquitectura de desplegament on es fa referència a eines SIC i no SIC proporcionades per CTTI per resoldre tots els serveis necessaris en aquest building block.


### 4.3 Disseny físic

El disseny físic identifica i descriu els components que formen part o integren el sistema, i com aquests es configuren o interactuen. Aquest disseny físic és la implementació del disseny lògic basat en cloud públic d’Azure.

![Disseny físic basat en Azure](/images/img30.png)

El disseny físic es basa en el disseny lògic basat en el cloud públic d'Azure. Dels serveis que ofereix Azure s'utilitzen sobre tot **Azure Kubernetes Service (AKS)**, **Grup de recurs**, **Application Gateway**, **Azure File Storage**, **Azure Cache**, **Azure Pipelines**, **Azure Repos**, **Azure Artifacts**, **Azure Container Registry**, **Azure Test Plans**, **Azure Key Vault**, **Azure Monitor** i **Azure App Insigths**. A més s'utilitzen **MongoDB Atlas** i **Confluent Cloud**.


## <a name="referencies"></a>Referències

El document complet en el qual es basa aquesta pàgina Web es pot descarregar com a fitxer PDF en ["Arquitectura de referència microserveis"](/references/ArquitecturaReferenciaMicro_V0.7.pdf).

#### Documentació complementària

* [AZ021] Microservices on Azure – What Is Microservices | Microsoft Azure [Internet]. [citat 18 març 2021]. Disponible a: https://azure.microsoft.com/en-us/solutions/microservice-applications/

* [AZU21] Improving observability of your Kubernetes deployments with Azure Monitor for containers [Internet]. [citat 18 març 2021]. Disponible a: https://azure.microsoft.com/en-us/blog/improving-observability-of-your-kubernetes-deployments-with-azure-monitor-for-containers/

* [BRA13] Brandolini A. Introducing Event Storming [Internet]. Ziobrando’s Lair. 2013 [citat 18 març 2021]. Disponible a: http://ziobrando.blogspot.com/2013/11/introducing-event-storming.html

* [CTT21] Principis d’arquitectura de sistemes d’informació [Internet]. [citat 18 març 2021]. Disponible a: https://canigo.ctti.gencat.cat/arqctti/principis_arq/

* [EVA03] Evans E. Domain-driven design: tackling complexity in the heart of software. Boston, MA: Addison-Wesley; 2003.

* [HAR21] Harbor [Internet]. [citat 18 març 2021]. Disponible a: https://goharbor.io/

* [HON21] The Microservices Observability Problem [Internet]. Honeycomb. [citat 18 març 2021]. Disponible a: https://www.honeycomb.io/microservices/

* [VER13] Vernon V. Implementing domain-driven design. Upper Saddle River, NJ; Mexico City: Addison-Wesley; 2013.

* [YOU79] Yourdon E, Constantine LL. Structured design: fundamentals of a discipline of computer program and systems design. Englewood Cliffs, N.J.: Yourdon Press; 1979.
