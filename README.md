# Import-dati-censuari-in-PostgreSQL/PostGIS

![GitHub last commit](https://img.shields.io/github/last-commit/ludovico85/import-dati-censuari-in-PostgreSQL-PostGIS?color=green&style=plastic)

Il repository contiene i passaggi per l'importazione dei dati catastali censuari in un database PostgreSQL/PostGIS e per il collegamento dei dati stessi alle geometrie catastali importate utilizzando il plugin per QGIS CXF_in https://github.com/saccon/CXF_in. Nella costruzione del database non sono state inserite esplicitamente delle relazioni tramite chiavi primarie/esterne a causa della non univocità dei valori presenti in alcuni tipi di file. Tuttavia le relazioni sono assicurate attarverso delle operazioni di join.

## 1. Breve descrizione dei dati catastali censuari
Le informazioni descritte in questa sezione derivano dal documento a cura dell'Agenzia dell'Entrate (DOC. ES-23-IS-05) liberamente consultabile all'indirizzo: https://wwwt.agenziaentrate.gov.it/mt/ServiziComuniIstituzioni/ES-23-IS-05_100909.pdf.

Per maggiori dettagli sui servizi riservati ai comuni di può consultare: https://www.agenziaentrate.gov.it/portale/web/guest/schede/fabbricatiterreni/portale-per-i-comuni/servizi-portale-dei-comuni/estrazione-dati-catastali.

I dati censuari sono costituiti da 4 tipi di file:

- file fabbricati (.FAB);

- file terreni (.TER);

- file soggetti (.SOG);

- file titolarità (.TIT);

- file parametri della richiesta (.PRM).

Ogni tipo di file è costituito da una tabella che può contenere diversi tipi di record. Il collegamento tra i tipi di file è assicurato dalla presenta di chiavi specifiche:

- .FAB/.TER contengono la chiave identificativo immobile;

- .SOG contiene la chiave identificativo soggetto;

- .TIT contiene sia la chiave identificativo immobile che la chiave identificativo soggetto (oltre che la chiave identificativo titolarietà);


## 2. Struttura del database
Per una gestione migliore della varie tabelle e viste che si andranno a creare è opportuno organizzare il database in schemi differenti:
- catasto_terreni = conterrà il dataset relativo al catasto terreni
- catasto_fabbricati = conterrà il dataset relativo al catasto fabbircati
- cxf_in = schema creato in automatico dal plug in cxf_in che conterrà i dati geografici

### 2.1. Schema catasto_terreni

**Tabelle**

- ter =  tabella derivata dal file .TER.
- sogp = tabella derivata dal file .SOG con TIPO SOGGETTO = 'P' (persona fisica).
- sogg = tabella derivata dal file .SOG con TIPO SOGGETTO = 'G' (persona giuridica).
- tit = tabella derivata dal file .TIT.
- codici_diritto = tabella contenente l'identificativo e la descrizione dei CODICI DIRITTO (pag. 36 del DOC. ES-23-IS-05).
- qualita = tabella contenente l'identificativo e la descrizione dei CODICI QUALITA' (pag. 38 del DOC. ES-23-IS-05).
- partite_speciali_terreni = tabella contenente l'identificativo e la descrizione delle PARTITE SPECIALI DEL CATASTO TERRENI (pag. 45 del DOC. ES-23-IS-05).
- partite_speciali_fabbricati = tabella contenente l'identificativo e la descrizione delle PARTITE SPECIALI DEL CATASTO FABBRICATI (pag. 45 del DOC. ES-23-IS-05).

**Viste**

- ter_1 = vista derivata dalla tabella ter contenente solo il TIPO RECORD = '1'.
- ter_1_clean = vista derivata dalla vista ter_1 ottenuta escludendo i codici PARTITA '0', '4', '5' e escludendo i subalterni.
- titg = vista derivata dalla tabella tit con TIPO SOGGETTO = 'P' (persona fisica).
- titg = vista derivata dalla tabella tit con TIPO SOGGETTO = 'G' (persona giuridica).
- titp_sogp = vista creata tramite join tra titp e sogp utilizzando il campo in comune identificativo_soggetto (persone fisiche).
- titp_sogp_json = vista ottenuta tramite l'aggregazione del campo identificativo_immobile dalla vista titp_sogp.
- titg_sogg = vista creata tramite join tra titg e sogg utilizzando il campo in comune identificativo_soggetto (persone giuridiche).
- titg_sogg_json = vista ottenuta tramite l'aggregazione del campo identificativo_immobile dalla vista titg_sogg.
- titp_sogp_ter_persone_fisiche = vista ottenuta tramite tramite join tra titp_sogp_json e la vista ter_1_clean utilizzando il campo in comune identificativo_immobile
(persone fisiche).
- titp_sogp_ter_persone_giuridiche = vista ottenuta tramite tramite join tra titg_sogg_json e la vista ter_1_clean utilizzando il campo in comune identificativo_immobile (persone giuridiche).
- particelle_partite_speciali_terreni = vista contenente le particelle senza titolairtà ottenuta tramite join tra la vista ter_1 e la tabella partite_speciali_terreni.

### 2.2. Schema catasto_fabbricati

**Tabelle**

**Viste**


 ## Schema cxf_in

**Tabelle**

- Acque = layer creato dal plugin cxf_in.
- Confine = layer creato dal plugin cxf_in.
- Fabbricati = layer creato dal plugin cxf_in.
- Fiduciali = layer creato dal plugin cxf_in.
- Linee = layer creato dal plugin cxf_in.
- Particelle = layer creato dal plugin cxf_in.
- Selezione = layer creato dal plugin cxf_in.
- Simboli = layer creato dal plugin cxf_in.
- Strade = layer creato dal plugin cxf_in.
- Testi = layer creato dal plugin cxf_in.

**Viste materializzate**

- particellare_persone_fisiche_mv: vista ottenuta tramite join tra titp_sogp_ter_persone_fisiche e il layer particelle utilizzando il campo in comune cm_fg_plla che identifica in modo univoco le particelle (persone fisiche). **Contiene le geometrie.**
- particellare_persone_giuridiche_mv: vista ottenuta tramite join tra titg_sogg_ter_persone_fisiche e il layer particelle utilizzando il campo in comune cm_fg_plla che identifica in modo univoco le particelle (persone giuridiche). **Contiene le geometrie.**
- particellare_partite_speciali_mv: vista ottenuta tramite join tra particelle_partite_speciali_terreni e il layer particelle utilizzando il campo in comune cm_fg_plla che identifica in modo univoco le particelle. **Contiene le geometrie.**


## 3. Impostazioni iniziali del database
Il pimo step riguarda la creazione degli schemi catasto_terreni e catasto_fabbricati:

```sql
CREATE SCHEMA catasto_terreni;
CREATE SCHEMA catasto_fabbricati;
```
Lo schema cxf_in viene costruito in automatico dal plugin cxf_in. In caso si voglia importare i cxf in altro modo:

```sql
CREATE SCHEMA cxf_in;
```

Bisogna installare l'estensione postgis:

```sql
CREATE EXTENSION postgis;
```

Per verificare quale sia lo schema corrente, nel quale si sta lavorando, bisogna interrogare il search_path:

```sql
SHOW search_path;
```

Di default lo schema è il public. Per abilitare lo schema nel quale si vuole lavorare:

```sql
SET search_path TO schema; -- sostituire con il nome dello schema
```


## 4. Elaborazione dei dati nello schema catasto_terreni
### 4.1. Importazione dei dati censuari in PostgreSQL/PostGIS
#### 4.1.2. Importazione della tabella .TER
Il file è costituito da 4 differenti tipi record. La particella è identificata attraverso il campo IDENTIFICATIVO IMMOBILE. La presenza di diversi tipi di record può creare delle righe duplicate per ogni particella.

- TIPO RECORD 1: contiene le caratteristiche della particella. E' il record di interesse che verrà utilizzato per ricostruire il dato spaziale;

- TIPO RECORD 2: deduzioni della particella;

- TIPO RECORD 3: riserva della particella;

- TIPO RECORD 4: porzioni della particella.

##### 4.1.1.1. Creazione della tabella .ter contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi)

```sql
SET search_path TO catasto_terreni; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public

CREATE TABLE ter(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER NOT NULL,
  tipo_immobile TEXT,
  progressivo INTEGER,
  tipo_record INTEGER,
  foglio TEXT,
  numero TEXT,
  denominatore TEXT,
  subalterno TEXT,
  edificialita TEXT,
  qualita TEXT,
  classe TEXT,
  ettari INTEGER,
  are INTEGER,
  centiare INTEGER,
  flag_reddito TEXT,
  flag_porzione TEXT,
  flag_deduzioni TEXT,
  reddito_dominicale_lire INTEGER,
  reddito_agrario_lire INTEGER,
  reddito_dominicale_euro REAL,
  reddito_agrario_euro REAL,
  data_efficacia_valore_atto TEXT,
  data_registrazione_atti_valore_atto TEXT,
  tipo_nota_valore_atto TEXT,
  numero_nota_valore_atto TEXT,
  progressivo_nota_valore_atto TEXT,
  anno_nota_valore_atto TEXT,
  data_efficacia_registrazione_atto TEXT,
  data_registrazione_registrazione_atto TEXT,
  tipo_nota_registrazione_atto TEXT,
  numero_nota_registrazione_atto TEXT,
  progressivo_nota_registrazione_atto TEXT,
  anno_nota_registrazione_atto TEXT,
  partita TEXT,
  annotazione TEXT,
  identificativo_mutazione_iniziale TEXT,
  identificativo_mutazione_finale TEXT,
  codice_causale_atto_generante TEXT,
  descrizione_atto_generante TEXT,
  codice_causale_atto_conclusivo TEXT,
  descrizione_atto_conclusivo TEXT
);
```
##### 4.1.1.2. Importazione dei dati
Convertire il file .TER in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/TER.csv). Inoltre è necessario sostituire il separatore decimale virgola (',') con il punto ('.'). Utilizzare un qualsiasi editor di testo (Notepad++, ATOM, ecc.).

#### 4.1.2. Importazione della tabella .TIT
Il file contiene un unico tipo di record.
##### 4.1.2.1. Creazione della tabella titolarità contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi).

```sql
CREATE TABLE tit
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione TEXT,
	identificativo_soggetto INTEGER NOT NULL,
	tipo_soggetto TEXT,
	identificativo_immobile INTEGER NOT NULL,
	tipo_immobile TEXT,
	codice_diritto TEXT,
	titolo_non_codificato TEXT,
	quota_numeratore_possesso INTEGER,
	quota_denominatore_possesso INTEGER,
	regime TEXT,
	soggetto_riferimento TEXT,
	data_validita_atto_generato TEXT,
	tipo_nota_generato TEXT,
	numero_nota_generato TEXT,
	progressivo_nota_generato TEXT,
	anno_nota_generato TEXT,
	data_registrazione_atti_generato TEXT,
	partita TEXT,
	data_validita_atto_concluso TEXT,
	tipo_nota_concluso TEXT,
	numero_nota_concluso TEXT,
	progessivo_nota_concluso TEXT,
	anno_nota_concluso TEXT,
	data_registrazione_atti_concluso TEXT,
	identificativo_mutazione_iniziale TEXT,
	identificativo_mutazione_finale TEXT,
	identificativo_titolarita INTEGER NOT NULL,
    codice_causale_atto_generante TEXT,
    descrizione_atto_generante TEXT,
    codice_causale_atto_conclusivo TEXT,
    descrizione_atto_conclusivo TEXT
);
```

##### 4.1.2.2. Importazione dei dati
Convertire il file .TIT in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/TIT.csv). Inoltre è necessario sostituire il separatore decimale virgola (',') con il punto ('.'). Utilizzare un qualsiasi editor di testo (Notepad++, ATOM, ecc.).

#### 4.1.3. Importazione della tabella .SOG
Il file è costituito da 2 differenti tipi record. Il soggetto è identificato attraverso il campo IDENTIFICATIVO SOGGETTO.

- TIPO RECORD P: intestato a persona fisica;

- TIPO RECORD G: intestato a persona giuridica;

Per una corretta gestione del file è necessario suddividere il file .SOG in due tabella, una per ogni record. Tale operazione si può effettuare in excel, calc, ecc.

##### 4.1.3.1. Creazione della tabella per i soggetti fisici sogP, contenente tutti i campi (Per non crare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi).

```sql
CREATE TABLE sogp
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione	TEXT,
	identificativo_soggetto	INTEGER NOT NULL,
	tipo_soggetto TEXT,
	cognome TEXT,
	nome TEXT,
	sesso TEXT,
	data_nascita TEXT,
	codice_amministratvio_comune_nascita TEXT,
	codice_fiscale	TEXT,
	indicazioni_supplementari TEXT
);
```

##### 4.1.3.2. Importazione dei dati
Convertire il file .SOG in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/SOGP.csv). Inoltre è necessario sostituire il separatore decimale virgola (',') con il punto ('.'). Utilizzare un qualsiasi editor di testo (Notepad++, ATOM, ecc.).

https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

##### 4.1.3.3. Creazione della tabella per i soggetti giuridici sogG, contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi).

```sql
CREATE TABLE sogg
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione TEXT,
	identificativo_soggetto  integer NOT NULL,
	tipo_soggetto TEXT,
	denominazione TEXT,
	codice_amministrativo_sede TEXT,
	codice_fiscale TEXT
);
```
##### 4.1.3.4. Importazione dei dati
Convertire il file .SOG in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/SOGG.csv)

#### 4.1.4. Creazione Tabelle aggiuntive
Le tabelle aggiuntive sono 3 e sono le qualità colturali, i codici di diritto e le partite speciali.
##### 4.1.4.1. Tabella delle qualità colturali
```sql
CREATE TABLE qualita
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	id_qualita TEXT,
	descrizione TEXT
);
```
Per inserire i valori utilizzare la funzione di PgAdmin per l'importazione dei CSV e utilizzare il file [qualita.csv](csv/qualita.csv) .
##### 4.1.4.2. Tabella dei codici di diritto
```sql
CREATE TABLE codici_diritto
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_diritto TEXT,
	descrizione TEXT
);
```
Per inserire i valori utilizzare la funzione di PgAdmin per l'importazione dei CSV e utilizzare il file [diritto.csv](csv/diritto.csv) .

##### 4.1.4.3 Tabella delle partite speciali
```sql
CREATE TABLE partite_speciali_terreni
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_partita TEXT,
	descrizione TEXT
);

INSERT INTO partite_speciali_terreni (codice_partita, descrizione)
VALUES
('1', 'aree di enti urbani e promiscui'),
('2', 'accessori comuni ad enti rurali e ad enti rurali e urbani'),
('3', 'aree di fabbricati rurali, o urbani da accertare, divisi in subalterni'),
('4', 'acque esenti da estimo'),
('5', 'strade pubbliche'),
('0', 'particelle soppresse')
```
### 4.2. Elaborazioni dei file
#### 4.2.1. Elaborazione della tabella .TER
Le elaborazioni consistono nell'assegnazione della descrizione della qualità colturale contenuta nella tabella qualità e nella pulizia delle particelle partite speciali.
##### 4.2.1.1. Aggiornamento della descrizione della qualità colturale
```sql
ALTER TABLE ter
ADD COLUMN descrizione_qualita TEXT;

UPDATE ter as t
SET descrizione_qualita = q.descrizione
FROM qualita as q
WHERE t.qualita = q.id_qualita

```
##### 4.2.1.2. Creazione della vista contenente solo il TIPO RECORD 1
Il tipo di record contiene la seguente codifica:
tipo di record | descrizione
:------ | :------
1   | contenente le informazioni descrittive della particella ed i suoi identificativi
2   | contenente le eventuali deduzioni
3   | contenente le riserve della particella
4   | contenente le porzioni della particella

```sql
CREATE OR REPLACE VIEW ter_1 AS
  SELECT * FROM ter
  WHERE tipo_record = '1';
```
##### 4.2.1.3. Pulizia della vista ter_1
La Vista risultante dalla selezione del tipo_record = '1' può essere ulteriormente "pulita" eliminando quelle particelle che non hanno titolarità come le particelle soppresse, acque, strade. Le informazioni possono essere ricavate dal campo partita:

partita | descrizione
:------ | :------
1   | aree di enti urbani e promiscui
2   | accessori comuni ad enti rurali e ad enti rurali e urbani
3   | aree di fabbircati rurali, o urbani da accertare, divisi in subalterni
4   | acque esenti da estimo
5   | strade pubbliche
0   | particelle soppresse

Altre particelle che possono essere "pulite" sono quelle che contengono i subalterni.

```sql
CREATE OR REPLACE VIEW ter_1_clean AS
	SELECT * FROM ter_1
	WHERE ter_1.partita IS DISTINCT FROM '0' AND ter_1.partita IS DISTINCT FROM '4' AND ter_1.partita IS DISTINCT FROM '5' AND subalterno IS NULL;
```

N.B. se nella richiesta dei dati le partite speciali non sono state scaricate è possibile saltare il passaggio (bisogna tuttavia eliminare i subalterni). Nel file .PRM è possibile verificare la Tipologia di esportazione che può essere 1) Terreni completa ptaspec no oppure 2) Terreni completa ptaspec si. In entrambi casi il passaggio della pulizia della vista ter_1 non modifica il risultato. Si consiglia comunque di eseguire il passaggio.

#### 4.2.2. Elaborazione della tabella .TIT
Le elaborazioni consistono nell'assegnazione della descrizione dei codici di diritto contenuti nella tabella codici_diritto e nella creazione di due distinte tabelle, per le persone fisiche e per le persone giuridiche.
##### 4.2.2.1 Aggiornamento della descrizione della qualità colturale
```sql
ALTER TABLE tit
ADD COLUMN descrizione_diritto TEXT;

UPDATE tit as t
SET descrizione_diritto = d.descrizione
FROM codici_diritto as d
WHERE
t.codice_diritto = d.codice_diritto
```
##### 4.2.2.2. Creazione della vista titolarità per i soggetti fisici

```sql
CREATE OR REPLACE VIEW titp AS
	SELECT * FROM tit
	WHERE tipo_soggetto = 'P';
```
##### 4.2.2.3. Creazione della vista titolarità per i soggetti giuridici
```sql
CREATE OR REPLACE VIEW titg AS
	SELECT * FROM tit
	WHERE tipo_soggetto = 'G';
```
### Creazione delle relazioni tra i tipi di file: soggetti persone fisiche (sogp) e titolarità persone fisiche (titp)
Ogni immobile (particella o fabbricato) può appartenere a più titolari. Ogni titolare può avere più immobili. Per gestire questa relazione (molti a molti) è possibile utilizzare le funzioni di aggregazione. In questo specifico caso è la scelta è ricaduta sulla creazione di un json che contiene i diversi titolari appartenenti ad un dato immobile. Il vantaggio di utilizzare il json è che questo è interrogabile. La creazione della relazione viene fatta in tre step.

#### 1) Creazione della vista titg_sogg_aggr. La relazione del tipo molti a molti viene esplicitata creando un campo univoco tra identificativo_immobile e idenitificativo_soggetto e tramite il join

```sql
CREATE OR REPLACE VIEW titp_sogp_aggr AS
SELECT
	row_number() OVER ()::integer AS gid,
	t.identificativo_immobile,
	t.tipo_immobile,
	t.identificativo_soggetto identificativo_soggetto_tit,
	t.descrizione_diritto as diritto,
	concat(t.quota_numeratore_possesso, '/', t.quota_denominatore_possesso) AS quota,
    p.identificativo_soggetto as identificativo_soggetto_sogp,
    p.cognome,
    p.nome,
    p.data_nascita,
    p.codice_fiscale,
    concat(t.identificativo_immobile, '_', t.identificativo_soggetto, '_', t.descrizione_diritto) AS immo_sogp_diritto
	FROM titp t
	JOIN sogp p ON t.identificativo_soggetto = p.identificativo_soggetto
```

#### 2) Creazione della vista titp_sogp_dist. Tramite il `SELECT DISTINCT ON` sul campo univoco, verranno selezionate solo le relazioni univoche eliminando eventuali records duplicati

```sql
CREATE OR REPLACE VIEW titp_sogp_dist AS
SELECT DISTINCT ON (immo_sogp_diritto) * FROM titp_sogp_aggr
```
#### 3) Creazione della vista aggregata. Viene creata la colonna soggetto che contiene in un'unica riga tutti i titolari dell'immobile

```sql
CREATE OR REPLACE VIEW titp_sogp_json AS SELECT
	row_number() OVER ()::integer AS gid,
	identificativo_immobile,
	json_agg
	(
		json_build_object
			(
				'identificativo_soggetto_sogp', identificativo_soggetto_sogp,
            	'cognome', cognome,
            	'nome', nome,
				'codice_fiscale', codice_fiscale,
				'data_nascita', data_nascita,
				'quota', quota,
				'diritto', diritto
			)
	) as soggetto
FROM titp_sogp_dist
GROUP by identificativo_immobile
```

### Creazione delle relazioni tra i tipi di file: soggetti giuridici (sogg) e titolarità soggetti giuridici (titg)
Ogni immobile (particella o fabbricato) può appartenere a più titolari e ogni titolare può avere più immobili. Per gestire questa relazione (molti a molti) è possibile utilizzare le funzioni di aggregazione. In questo specifico caso è la scelta è ricaduta sulla creazione di un json che contiene i diversi titolari appartenenti ad un dato immobile. Il vantaggio di utilizzare il json è che questo è interrogabile. La creazione della relazione viene fatta in tre step.

#### 1) Creazione della vista titg_sogg_aggr. La relazione del tipo molti a molti viene esplicitata creando un campo univoco tra identificativo_immobile e idenitificativo_soggetto e tramite il join

```sql
CREATE OR REPLACE VIEW titg_sogg_aggr AS
SELECT
	row_number() OVER ()::integer AS gid,
	t.identificativo_immobile,
	t.tipo_immobile,
	t.identificativo_soggetto as identificativo_soggetto_tit,
	t.descrizione_diritto as diritto,
	concat(t.quota_numeratore_possesso, '/', t.quota_denominatore_possesso) AS quota,
    g.identificativo_soggetto as identificativo_soggetto_sogg,
	g.denominazione,
	g.codice_amministrativo_sede,
	g.codice_fiscale,
    concat(t.identificativo_immobile, '_', t.identificativo_soggetto, '_', t.descrizione_diritto) AS immo_sogg_diritto
	FROM titg t
	JOIN sogg g ON t.identificativo_soggetto = g.identificativo_soggetto
```
#### 2) Creazione della vista titg_sogg_dist. Tramite il `SELECT DISTINCT ON` sul campo univoco, verranno selezionate solo le relazioni univoche eliminando eventuali records duplicati
```sql
CREATE OR REPLACE VIEW titg_sogg_dist AS
SELECT DISTINCT ON (immo_sogg_diritto) * FROM titg_sogg_aggr
```
#### 3) Creazione della vista aggregata. Viene creata la colonna soggetto che contiene in un'unica riga tutti i titolari dell'immobile
```sql
CREATE OR REPLACE VIEW titg_sogg_json AS SELECT
	row_number() OVER ()::integer AS gid,
	identificativo_immobile,
	json_agg
	(
		json_build_object
			(
				'identificativo_soggetto_sogg', identificativo_soggetto_sogg,
				'denominazione', denominazione,
            	'codice_amministrativo_sede', codice_amministrativo_sede,
				'codice_fiscale', codice_fiscale,
				'quota', quota,
				'diritto', diritto
			)
	) as soggetto
FROM titg_sogg_dist
GROUP by identificativo_immobile;
```
### Creazione delle relazioni tra i tipi di file: soggetti_titolarità_persone_fisiche (titp_sogp_json) e immobili (ter_1_clean)
```sql
CREATE OR REPLACE VIEW titp_sogp_ter_persone_fisiche AS
SELECT row_number() OVER ()::integer AS gid,
ter.identificativo_immobile AS identificativo_immobile,
ter.foglio,
ter.numero,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per il join con le geometrie del catasto
	WHEN length(ter.foglio) = 1 THEN concat(ter.codice_amministrativo, '_000', ter.foglio, '_', ter.numero)
    	ELSE
		(
			CASE
		 	WHEN length(ter.foglio) = 2 THEN concat(ter.codice_amministrativo, '_00', ter.foglio, '_', ter.numero)
		 	ELSE
				(
					CASE
			 		WHEN length(ter.foglio) = 3 THEN concat(ter.codice_amministrativo, '_0', ter.foglio, '_', ter.numero)
			 		ELSE
			 			(
							CASE
							WHEN length(ter.foglio) = 4 THEN concat(ter.codice_amministrativo, '_', ter.foglio, '_', ter.numero)
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
ter.descrizione_qualita AS qualita,
ter.classe,
ter.ettari,
ter.are,
ter.centiare,
j.soggetto
FROM ter_1_clean ter
JOIN titp_sogp_json j ON ter.identificativo_immobile = j.identificativo_immobile;
```
### Creazione delle relazioni tra i tipi di file: soggetti_titolarità persone giuridiche (titg_sogg_json) e immobili (ter_1_clean)
```sql
CREATE OR REPLACE VIEW titg_sogg_ter_persone_giuridiche AS
SELECT row_number() OVER ()::integer AS gid,
ter.identificativo_immobile AS identificativo_immobile,
ter.foglio,
ter.numero,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per il join con le geometrie del catasto
	WHEN length(ter.foglio) = 1 THEN concat(ter.codice_amministrativo, '_000', ter.foglio, '_', ter.numero)
    	ELSE
		(
			CASE
		 	WHEN length(ter.foglio) = 2 THEN concat(ter.codice_amministrativo, '_00', ter.foglio, '_', ter.numero)
		 	ELSE
				(
					CASE
			 		WHEN length(ter.foglio) = 3 THEN concat(ter.codice_amministrativo, '_0', ter.foglio, '_', ter.numero)
			 		ELSE
			 			(
							CASE
							WHEN length(ter.foglio) = 4 THEN concat(ter.codice_amministrativo, '_', ter.foglio, '_', ter.numero)
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
ter.descrizione_qualita AS qualita,
ter.classe,
ter.ettari,
ter.are,
ter.centiare,
j.soggetto
FROM ter_1_clean ter
JOIN titg_sogg_json j ON ter.identificativo_immobile = j.identificativo_immobile;
```
## Elaborazione dei dati nello schema catasto_fabbricati

### Importazione dei singoli file in PostgreSQL/PostGIS
#### Importazione della tabella .FAB
Il file è costituito da 5 differenti tipi di records. Il fabbricato è distinto tramite il campo IDENTIFICATIVO IMMOBILE. La presenza di diversi tipi di record può creare delle righe duplicate per ogni fabbricato.
- TIPO DI RECORD 1: contiene le informazioni descrittive dell'unità immobiliare
- TIPO DI RECORD 2: contiene gli identificativi dell'unità immobiliare
- TIPO DI RECORD 3: contiene gli indirizzi dell'unità immobiliare
- TIPO DI RECORD 4: contiene le unità comuni dell'unità immobiliare
- TIPO DI RECORD 5: contiene le riserve dell'unità immobiliare

Prima di importare il dato è necessario separare i tipi di record (1,2,3,4,5) in altrettante tabelle (es. fab1, fab2, fab3, fab4 e fab5) in modo da non creare confusione nell'importazione del dato. L'operazione può essere fatta in excel, calc, ecc.

ATTENZIONE CARATTERI SPECIALI NON RICONOSCIUTI: °, SOSTITUIRE LA VIRGOLA CON IL PUNTO

##### Tabella fab_1
```sql
SET search_path TO catasto_fabbricati;
CREATE TABLE fab_1(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER,
  tipo_immobile TEXT,
  progressivo INTEGER,
  tipo_record INTEGER,
  zona TEXT,
  categoria TEXT,
  classe TEXT,
  consistenza REAL,
  superficie INTEGER,
  rendita_lire REAL,
  rendita_euro REAL,
  lotto TEXT,
  edificio TEXT,
  scala TEXT,
  interno_1 TEXT,
  interno_2 TEXT,
  piano_1 TEXT,
  piano_2 TEXT,
  piano_3 TEXT,
  piano_4 TEXT,
  data_efficacia_generato TEXT,
  data_registrazione_atti_in_atti_dal TEXT,
  tipo_nota TEXT,
  numero_nota TEXT,
  progressivo_nota TEXT,
  anno_nota  TEXT,
  data_efficacia_concluso TEXT,
  data_resgitrazione_atti_ufficio TEXT,
  tipo_nota_registrazione TEXT,
  numero_nota_registrazione TEXT,
  progressivo_nota_registrazione TEXT,
  anno_nota_registrazione TEXT,
  partita TEXT,
  annotazione TEXT,
  identificativo_mutazione_iniziale TEXT,
  identificativo_mutazione_finale TEXT,
  protocollo_notifica TEXT,
  data_notifica TEXT,
  codice_causale_atto_generante TEXT,
  descrizione_atto_generante TEXT,
  codice_causale_atto_conclusivo TEXT,
  descrizione_atto_conclusivo TEXT,
  flag_classamento TEXT);
```
##### Tabella fab_2

```sql
CREATE TABLE fab_2(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER,
  tipo_immobile TEXT,
  progressivo INTEGER,
  tipo_record INTEGER,
  sezione_urbana TEXT,
  foglio TEXT,
  numero TEXT,
  denominatore TEXT,
  subalterno TEXT,
  edificialita TEXT);
```
##### Tabella fab_3

```sql
CREATE TABLE fab_3(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER,
  tipo_immobile TEXT,
  progressivo INTEGER,
  tipo_record INTEGER,
  toponimo TEXT,
  indirizzo TEXT,
  civico_1 TEXT,
  civico_2 TEXT,
  civico_3 TEXT,
  codice_strada TEXT);
```

##### Tabella fab_4

```sql
CREATE TABLE fab_4(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER,
  tipo_immobile TEXT,
  progressivo INTEGER,
  tipo_record INTEGER,
  foglio TEXT,
  numero TEXT,
  denominatore TEXT,
  subalterno TEXT);
```

##### Tabella fab_5

```sql
CREATE TABLE fab_5(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER,
  tipo_immobile TEXT,
  codice_riserva INTEGER,
  partita_iscrizione_riserva INTEGER);
```

##### Tabella partite speciali fabbricati
```sql
CREATE TABLE partite_speciali_fabbricati
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_partita TEXT,
	descrizione TEXT
);

INSERT INTO partite_speciali_fabbricati (codice_partita, descrizione)
VALUES
('0', 'beni comuni censibili'),
('A', 'beni comuni non censibili'),
('R', 'fabbricati rurali'),
('C', 'unità immobiliari soppresse');
```
#### Creazione tabella categorie catastali
``` sql
CREATE TABLE categoria_catastale
(
    pk_id integer NOT NULL GENERATED BY DEFAULT AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    codice_categoria TEXT,
	descrizione_categoria TEXT
);
INSERT INTO categoria_catastale (codice_categoria, descrizione_categoria)
VALUES
('A08','abitazioni in ville'),
('A09','castelli, palazzi di eminenti pregi artistici e storici'),
('A10','uffici e studi privati'),
('A11','abitazioni ed alloggi tipici dei luoghi'),
('B01','collegi, convitti; educandati, ricoveri orfanatrofi, ospizi, conventi, seminari, caserme'),
('B02','case di cura ed ospedali senza fini di lucro'),
('B03','prigioni e riformatori'),
('B04','uffici pubblici'),
('B05','scuole e laboratori scientifici'),
('B06','biblioteche, pinacoteche, musei, gallerie, accademie, circoli ricreativi e culturali senza fine di lucro, che non hanno sede in edifici della categoria A/9'),
('B07','cappelle ed oratori non destinati all’esercizio pubblico dei culti'),
('B08','magazzini sotterranei per deposito derrate'),
('C01','negozi e botteghe'),
('C02','magazzini e locali di deposito; cantine e soffitte se non unite all`unità immobiliare abitativa'),
('C03','laboratori per arti e mestieri'),
('C04','fabbricati e locali per esercizi sportivi'),
('C05','stabilimenti balneari e di acque curative'),
('C06','stalle, scuderie, rimesse ed autorimesse'),
('C07','tettoie; posti auto su aree private; posti auto coperti'),
('D01','opifici'),
('D02','alberghi e pensioni'),
('D03','teatri, cinematografi, sale per concerti e spettacoli; arene, parchi giochi, zoo-safari'),
('D04','case di cura e ospedali'),
('D05','istituti di credito, cambio ed assicurazione'),
('D06','fabbricati, locali, aree attrezzate per esercizi sportivi'),
('D07','fabbricati costruiti o adattati per le speciali esigenze di un`attivita` industriale e non suscettibile di destinazione diversa senza radicali trasformazioni'),
('D08','fabbricati costruiti o adattati per le speciali esigenze di un`attivita` commerciale e non suscettibili di destinazione diversa senza radicali trasformazioni'),
('D09','edifici galleggianti o assicurati a punti fissi del suolo; ponti privati soggetti a pedaggio; aree attrezzate per l’appoggio di palloni aerostatici e dirigibili'),
('D10','fabbricati per funzioni produttive connesse alle attività agricole'),
('E01','stazioni per servizi di trasporto terrestri, marittimi ed aerei; stazioni per metropolitane; stazioni per ferrovie; impianti di risalita in genere'),
('E02','ponti comunali e provinciali soggetti a pedaggio'),
('E03','costruzioni e fabbricati per speciali esigenze pubbliche'),
('E04','recinti chiusi per mercati, fiere, posteggio bestiame e simili'),
('E05','fabbricati costituenti fortificazioni e loro dipendenze'),
('E06','fari, semafori torri per rendere pubblico l’uso dell’orologio comunale'),
('E07','fabbricati per l’esercizio pubblico dei culti'),
('E08','fabbricati e costruzioni nei cimiteri esclusi i colombari, i sepolcri e le tombe di famiglia'),
('E09','edifici a destinazione particolare non compresi nelle categorie precedenti del gruppo E'),
('F01','area urbana'),
('F02','unità collabenti'),
('F03','unità in corso di costruzione'),
('F04','unità in corso di definizione'),
('F05','lastrico solare');
```
##### Creazione tabella codice strada
```sql
CREATE TABLE codice_strada (
pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
codice_strada INTEGER,
denominazione TEXT
);
INSERT INTO codice_strada (codice_strada, denominazione) VALUES
    (0, ''),
    (54, 'Contrada'),
	(58, 'Corso'),
	(86, 'Largo'),
	(90, 'Località'),
	(130, 'Piazza'),
	(172, 'Salita'),
	(210, 'Strada'),
	(224, 'Traversa'),
	(236, 'Via'),
	(240, 'Viale'),
	(244, 'Vico'),
	(546, 'Strada comunale'),
	(566, 'Strada provinciale'),
	(661, 'Vico chiuso'),
	(894, 'Strada poderale');
```

### Importazione dei singoli file in PostgreSQL/PostGIS - .SOG (in costruzione).
### Importazione dei singoli file in PostgreSQL/PostGIS - .TIT (in costruzione).
### Creazione delle tabella aggiuntive per la codifica dei codici (in costruzione).
### Creazione delle relazioni tra i tipi di file: soggetti persone fisicihe (sogp) e titolarità persone fisiche (titp) (in costruzione).
### Creazione delle relazioni tra i tipi di file: soggetti giuridici (sogg) e titolarità soggetti giuridici (titg) (in costruzione).
### Creazione delle relazioni tra i tipi di file: soggetti_titolarità persone fisiche (titp_sogp_json) e FABBRICATI (in costruzione).
### Creazione delle relazioni tra i tipi di file: soggetti_titolarità persone giuridiche (titg_sogg_json) e FABBRICATI (in costruzione).


## Elaborazione dei dati nello schema cxf_in
Il plugin CXF_in importa i dati nel databse PostgreSQL/PostGIS nello schema cxf_in. Se si vuole, si possono copiare le tabelle in un altro schema (in questo modo consente di avere anche una copia di backup dei dati importati).
**Nota bene: può succedere che l'importazione nel DB attraverso il plugin CXF_in generi degli errori a causa di geometrie non valide, interrompendo il processo. In tal caso occorre importare dapprima le geometrie nel db spatialite (in modo da consentire la georeferenzazione), caricarle in QGIS, ripararle (tramite lo strumento ripara geometrie) e importarle in PostgreSQL/PostGIS (utilizzando il tool Esporta in PostgreSQL).**

![Error](/img/Error_PostGIS.JPG)

#### Copia delle tabelle in altro schema (OPZIONALE)

```sql
CREATE TABLE schema.acque AS (SELECT * FROM cxf_in.acque); -- sostituire il nome dello schema con quello desiderato (es. public).
CREATE TABLE schema.confine AS (SELECT * FROM cxf_in.confine);
CREATE TABLE schema.fabbricati AS (SELECT * FROM cxf_in.fabbricati);
CREATE TABLE schema.fiduciali AS (SELECT * FROM cxf_in.fiduciali);
CREATE TABLE schema.linee AS (SELECT * FROM cxf_in.linee);
CREATE TABLE schema.particelle AS (SELECT * FROM cxf_in.particelle);
CREATE TABLE schema.selezione AS (SELECT * FROM cxf_in.selezione);
CREATE TABLE schema.simboli AS (SELECT * FROM cxf_in.simboli);
CREATE TABLE schema.strade AS (SELECT * FROM cxf_in.strade);
CREATE TABLE schema.testi AS (SELECT * FROM cxf_in.testi);
```

### Creazione delle relazioni tra le geometrie particellari e i dati censuari (persone fisiche)

#### 1) Creazione dell'identificativo univoco di particella da utilizzare nel join con la vista tit_sog_ter_persone_fisiche
**N.B. Se le geometrie sono state importate attraverso il tool Esporta in PostgreSQL di QGIS, è necessario specificare, negli script seguenti, la tabella Particelle tra doppi apici ("Particelle") ed adattare i nomi dei campi**

```sql
SET search_path TO cxf_in; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public;

ALTER TABLE Particelle
ADD COLUMN com_fg_plla TEXT;

UPDATE Particelle
SET com_fg_plla = CONCAT(codice_comune, '_',fg,'_', mappale);
```

#### 2) Join delle informazioni delle particelle e della titolarità relative ai soggetti fisici

```sql
CREATE OR REPLACE VIEW particellare_persone_fisiche AS -- Per questioni di performance sostituire con una Materialized View
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titp_sogp_ter_persone_fisiche j ON p.com_fg_plla = j.com_fg_plla
```

**In alternativa con una materialized view (Consigliato)**

```sql
CREATE MATERIALIZED VIEW particellare_persone_fisiche_MV AS
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titp_sogp_ter_persone_fisiche j ON p.com_fg_plla = j.com_fg_plla
WITH DATA
```

### Creazione delle relazioni tra le geometrie particellari e i dati censuari (persone giuridiche)

#### 1) Join delle informazioni delle particelle e della titolarità relative ai soggetti giuridici

```sql
CREATE OR REPLACE VIEW particellare_persone_giuridiche AS -- Per questioni di performance sostituire con una Materialized View
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titg_sogg_ter_persone_giuridiche j ON p.com_fg_plla = j.com_fg_plla
```

**In alternativa con una materialized view (Consigliato)**

```sql
CREATE MATERIALIZED VIEW particellare_persone_giuridiche_MV AS
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titg_sogg_ter_persone_giuridiche j ON p.com_fg_plla = j.com_fg_plla
WITH DATA
```
### Particelle senza titolarità
La tabella partite_speciali_terreni contiene le particelle che non hanno titolarità. Può tornare utile creare un layer geometrico distinto per tale categoria di particelle.
#### 1) Selezione e creazione della vista con la partite speciali terreni
```sql
SET search_path TO catasto_terreni; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public;

CREATE OR REPLACE VIEW particelle_partite_speciali_terreni AS
SELECT ter_1.*,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per il join con le geometrie del catasto
	WHEN length(ter_1.foglio) = 1 THEN concat(ter_1.codice_amministrativo, '_000', ter_1.foglio, '_', ter_1.numero)
    	ELSE
		(
			CASE
		 	WHEN length(ter_1.foglio) = 2 THEN concat(ter_1.codice_amministrativo, '_00', ter_1.foglio, '_', ter_1.numero)
		 	ELSE
				(
					CASE
			 		WHEN length(ter_1.foglio) = 3 THEN concat(ter_1.codice_amministrativo, '_0', ter_1.foglio, '_', ter_1.numero)
			 		ELSE
			 			(
							CASE
							WHEN length(ter_1.foglio) = 4 THEN concat(ter_1.codice_amministrativo, '_', ter_1.foglio, '_', ter_1.numero)
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
p.descrizione as descrizione_partita
FROM ter_1
JOIN partite_speciali_terreni p ON ter_1.partita = p.codice_partita
WHERE ter_1.partita IN ('1', '2', '3', '4', '5', '0')
```
#### 2) Creazione delle geometrie
```sql
SET search_path TO cxf_in; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public;

CREATE MATERIALIZED VIEW particellare_partite_speciali_mv AS
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS fg,
	p.mappale as plla,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.*,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.particelle_partite_speciali_terreni j ON p.com_fg_plla = j.com_fg_plla
WITH DATA
```
### Estrazione delle particelle appartenenti ad un determinato soggetto.
Si vogliono estrarre, per esempio, le particelle di un dato comune. Bisogna interrogare il campo soggetto (che è un json array):
```sql
SELECT * FROM particellare_persone_giuridiche_mv WHERE (soggetto::jsonb @> '[{"denominazione": "VALORE"}]'); -- sotituire a VALORE il valore desiderato (es. COMUNE DI XXXX)
```
Per creare il particellare con il solo soggetto:
```sql
CREATE MATERIALIZED VIEW particellare_SOGGETTO_mv AS
SELECT * FROM particellare_persone_giuridiche_mv WHERE (soggetto::jsonb @> '[{"denominazione": "VALORE"}]') -- sotituire a VALORE il valore desiderato (es. COMUNE DI XXXX)
WITH DATA
```
## Verifica delle particelle
Nel file titolarità sono riportati i codici univoci degli immobili e dei titolari. Per conoscere quante particelle sono riportare nel file titolarità:
```sql
SELECT DISTINCT identificativo_immobile FROM tit WHERE tipo_immobile = 'T'
```
Per conoscere quante sono le particelle la cui titolerità riguarda soggetti fisici:

```sql
SELECT DISTINCT identificativo_immobile FROM tit WHERE tipo_immobile = 'T' AND tipo_soggetto = 'P'
```
Per conoscere quante sono le particelle la cui titolerità riguarda soggetti giuridici:
```sql
SELECT DISTINCT identificativo_immobile FROM tit WHERE tipo_immobile = 'T' AND tipo_soggetto = 'G'
```sql
La somma del numero delle particelle soggetti fisici e del numero delle particelle soggetti giuridici non è sempre uguale al numero totale degli immobili poiché alcune particelle potrebbero essere in comune tra i due gruppi.
```

## Gestione dei dati in QGIS
### Catasto fabbricati
La gestione dei dati in QGIS può avvenire in maniera semplificata utilizzando le relazioni tra le particelle e l'identificativo dell'immobile.

1) Creazione della Vista univoca con i dati delle titolarità dei soggetti giuridici, delle persone fisiche e delle partite speciali.

```sql
set search_path TO catasto_terreni;
CREATE OR REPLACE VIEW titolarita_partite_speciali_union AS
SELECT
g.identificativo_immobile as identificativo_immobile,
g.tipo_immobile as tipo_immobile,
'soggetto giuridico' as tipo_soggetto,
g.diritto as diritto,
g.quota as quota,
g.identificativo_soggetto_sogg as identificativo_soggetto,
g.denominazione as denominazione,
g.codice_amministrativo_sede as codice_amministrativo_sede,
NULL as data_nascita,
g.codice_fiscale as codice_fiscale,
g.immo_sogg_diritto as idimm_idsog_diritto
FROM catasto_terreni.titg_sogg_dist g
UNION ALL
SELECT
p.identificativo_immobile as identificativo_immobile,
p.tipo_immobile as tipo_immobile,
'persona fisica' as tipo_soggetto,
p.diritto as diritto,
p.quota as quota,
p.identificativo_soggetto_sogp as identificativo_soggetto,
concat(p.cognome, ' ', p.nome) as denominazione,
NULL as codice_amministrativo_sede,
p.data_nascita as data_nascita,
p.codice_fiscale as codice_fiscale,
p.immo_sogp_diritto as idimm_idsog_diritto
FROM catasto_terreni.titp_sogp_dist p
UNION ALL
SELECT
s.identificativo_immobile as identificativo_immobile,
s.tipo_immobile as tipo_immobile,
'partita speciale' as tipo_soggetto,
NULL as diritto,
NULL as quota,
NULL as identificativo_soggetto,
s.descrizione_partita as denominazione,
NULL as codice_amministrativo_sede,
NULL as data_nascita,
NULL as codice_fiscale,
NULL as idimm_idsog_diritto
FROM catasto_terreni.particelle_partite_speciali_terreni s
WHERE s.descrizione_partita NOT IN ('particelle soppresse');
```
2) Creazione della relazione con la tabella .ter_1_clean

```sql
CREATE OR REPLACE VIEW titolarita_partite_speciali_union_ter AS
SELECT row_number() OVER ()::integer AS gid,
ter.identificativo_immobile AS identificativo_immobile_ter,
ter.foglio,
ter.numero,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per il join con le geometrie del catasto
	WHEN length(ter.foglio) = 1 THEN concat(ter.codice_amministrativo, '_000', ter.foglio, '_', ter.numero)
    	ELSE
		(
			CASE
		 	WHEN length(ter.foglio) = 2 THEN concat(ter.codice_amministrativo, '_00', ter.foglio, '_', ter.numero)
		 	ELSE
				(
					CASE
			 		WHEN length(ter.foglio) = 3 THEN concat(ter.codice_amministrativo, '_0', ter.foglio, '_', ter.numero)
			 		ELSE
			 			(
							CASE
							WHEN length(ter.foglio) = 4 THEN concat(ter.codice_amministrativo, '_', ter.foglio, '_', ter.numero)
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
ter.descrizione_qualita AS qualita,
ter.classe,
ter.ettari,
ter.are,
ter.centiare,
t.*,
concat(ter.identificativo_immobile, '_',t.identificativo_immobile,'_', t.identificativo_soggetto, '_', t.diritto) as univoco
FROM ter_1_clean ter
RIGHT JOIN titolarita_partite_speciali_union t ON ter.identificativo_immobile = t.identificativo_immobile;
```

---

## EXTRA
### Estrazione del geojson utilizzando ogr2ogr
```
ogr2ogr -f GeoJSON out.json "PG:host=myhost dbname=mydb user=ubuntu password=mypassword" \ -sql "select * from table"
```
### Verifica del Sistema di coordinate
```sql
SELECT ST_SRID(geom) FROM particellare_proprieta_comunale LIMIT 1;
```
### Riproiezione
```sql
CREATE MATERIALIZED VIEW particellare_proprieta_comunale_4326 AS
SELECT gid, codice_comune, foglio, particella, fg_plla, qualita, classe, ettari, are, centiare, soggetto,
	ST_Transform(geom,4326)
FROM
	particellare_proprieta_comunale
WITH DATA
```
