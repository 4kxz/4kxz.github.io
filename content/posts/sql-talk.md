+++
title = 'Conceptes de SQL'
date = 2021-03-16T13:14:09+02:00
draft = false
tags = ["SQL"]
+++

## Definicions

* **Servidor**: la base de dades.
* **Client**: el teu programa.
* **Driver**: és el software/llibreria que es fa servir per connectar a la base de dades. Normalment cada llenguatge de programació té un driver per cada base de dades.
* **Connexió**: és l'estructura de dades que crea el client per gestionar la comunicació amb la base de dades. Normalment li pases adreça, user i pass i fas un connect i et retorna una connexió. A la connexió li passes les queries i al acabar s'ha de fer un close.
* **Cursor**: és l'estructura de dades que genera el client per recuperar els resultats d'una query.
* **Transacció**: és la unitat de treball amb la base de dades.
* **ORM** (Object-Relational Mapper): llibreria que adapta les estructures d'una base de dades relacional per usar-les amb orientació a objectes.



## Transaccions

Les operacions sobre una base de dades SQL es fan sempre dins d'una **transacció**.
Una transacció s'inicia, fa una serie d'operacions de lectura i/o escriptura i es tanca fent un **commit**.
En qualsevol punt es pot fer un **rollback**, desfent tots els canvis desde l'inici de la transacció.

Moltes bases de dades permeten usar **autocommit**, que fa una transacció per cada query de manera implicita.
Autocommit és ineficient, pero no cal estar pendent de fer begin/commit/rollback per tot el codi.


### ACID
Les transaccions són un mecanisme per garantir les propietats **ACID**:

* **Atomicity**: Les operacions d'una transacció es tracten en bloc, s'apliquen totes o cap.
* **Consistency**: Totes les restriccions (tipus de dades, keys, constraints, etc.) s'han de complir al fer commit.
* **Isolation**: Els canvis d'una transaccion no afecten a altres transaccions.
* **Durability**: Un cop fet commit les dades perduren, tota transacció posterior veurà les dades actualitzades.

En sistemes grans amb interaccions complexes aquestes propietats ajuden a evitar inconsistencies, [heisenbugs](https://en.wikipedia.org/wiki/Heisenbug) i sorpreses desagradables.
Garantir-les requereix fer comprovacions i bloquejos que alenteixen les queries.
Per aprofitar-les al màxim cal que el model de dades estigui ben definit amb les restriccions ben posades.


### Concurrencia
Mantenir les propietats ACID és fàcil quan tenim queries seqüencials, però la majoria de bases de dades modernes permeten fer **queries concurrents**. Això entra en conflicte amb la propietat de isolation, donant peu a problemes com els següents:

#### Dirty read
Una transacció ha llegit un valor que ja no és vàlid perquè s'ha fet un rollback.

![Dirty read](http://i.imgur.com/TRWdgIp.jpg)

#### Non-repeatable read
Una transacció llegeix dos cops i obté resultats diferents perquè una altra transacció el modifica.

![Non-repeatable read](http://i.imgur.com/NRZ0R9g.gif)

#### Phantom read
Es queden elements sense llegir perquè s'han commitejat més tard.

![Phantom read](http://i.imgur.com/nOTkg9s.gif)


### Isolation levels
SQL defineix quatre isolation levels, que permeten triar un compromis entre isolation i eficiència:

* **READ UNCOMMITTED**: Va a saco.
* **READ COMMITTED**: La transacció espera per llegir a que les escriptures acabin.
* **REPEATABLE READ**: Espera com l'anterior però un cop llegit ningú pot escriure fins que acabi.
* **SERIALIZABLE**: Espera, un cop llegit ningú pot escriure, inserir ni eliminar.

El nivell es pot indicar al començar la transacció. A Postgres per defecte és READ COMMITTED.

Taula amb els nivells d'isolation i quin és el problema que eviten:

| Isolation level | Dirty reads | Non-repeatable reads | Phantoms |
|-----------------|-------------|----------------------|----------|
| READ UNCOMMITTED | may occur | may occur | may occur |
| READ COMMITTED | - | may occur | may occur |
| REPEATABLE READ | - | - | may occur |
| SERIALIZABLE | - | - | - |

Evitar els problemes implica fer servir *locks* per les modificacions i aleshores poden quedar queries aturades fins que la que té el lock faci un commit o rollback.


### NoSQL

Normalment les bases de dades NoSQL passen del ACID i estàn optimitzades per anar més ràpid fent queries.



## Dades

### Estructura

#### Cluster
* Quan instales PostgreSQL es crea un cluster, cada instància Amazon RDS és un cluster.
* Quan actualitzes a una versió nova, has d'actualitzar el cluster.
* Els backups de RDS i restaurar snapshots es fa tot a nivell de cluster.

#### Database
* N'hi pot haver més d'una en un cluster.
* Els usuaris son els mateixos per totes les databases. Però es poden assignar permisos.
* S'ha d'obrir una connexió per cada base de dades a la que vulguis fer queries.
* Potser hi ha alguna comanda per canviar de db, pero segur que no es poden fer queries a dos dbs diferents alhora.
* Les dades estan separades, no es pot fer join de dues taules si estan en dbs diferents.
* Amb Amazon RDS és una mica redundant, pero va bé fer varies DBs quan només tens un servidor físic.

#### Schema
* Serveix només a nivell organitzatiu, són com namespaces. Pots fer un schema per llum, un per gas, i tenir-ho tot ordenat.
* Equivalent a les databases MySQL.
* Es poden fer joins entre dues taules a schemas diferents.
* Es poden moure taules d'un schema a l'altra fàcilment.
* Si no s'indica un schema explicitament, tot va a parar al schema per defecte que es diu "public".
* El schema conté tables, functions, views...

#### Taules
* Una taula te definides una serie de columnes i cada columna té un tipus.

#### Permisos
* Els permisos es poden posar a nivell de database, schema o table.
* S'assignen a un role, que és com es representa un usuari. Un grup d'usuaris també es representa amb un role.


### Tipus de dades

És important triar el tipus més adequat per cada columna ja que pot tenir un impacte gran sobre el rendiment. Per exemple, si totes les columnes són de tipus integer i n'hi ha prou amb smallint, les queries poden anar casi el doble de ràpid.

A la documentació oficial expliquen cada tipus, i quins valors representen:

* https://www.postgresql.org/docs/9.5/static/datatype.html

#### Integer
Aquest és fàcil: s'ha de mirar el rang de valors representables i triar el més adequat:

| Tipus | Mida | Rang de dades |
|-------|------|---------------|
| smallint | 2 bytes | -32768 to +32767 |
| integer | 4 bytes | -2147483648 to +2147483647 |
| bigint | 8 bytes | -9223372036854775808 to +9223372036854775807 |

#### Float
| Tipus | Mida | Rang de dades |
|-------|------|---------------|
| Real | 4 bytes | Precisió variable, uns 6 decimals |
| Double | 8 bytes | Precisió variable, uns 15 decimals |
| Decimal | variable | Permet triar la precisió |

#### Text
El tipus TEXT es pot usar tranquilament per tot, té mida variable ilimitada i coses útils com full text search.

El tipus VARCHAR és un TEXT amb mida màxima.

El tipus CHAR té mida fixa. És recomanable per gastar espai.

#### Dates
Hi ha els tipus date, time i timestamp. El timestamp pot ser amb o sense timezone.

**LES DATES SEMPRE S'HAN DE GUARDAR EN UTC.**

És una regla sagrada i si algú no ho compleix es mereix que li tallin una mà.

Si mai trobeu una base de dades on els timestamps no estan en UTC, heu de seguir el procediment següent:

* Pregunteu per la persona responsable d'administrar les bases de dades.
* Localitzeu la seva posició a la oficina.
* Sortiu corrents en la direcció oposada, tan ràpid com us sigui físicament possible.
* Passi el que passi, no mireu enrera.

#### Array
Super útil quan un valor és una tupla, com la potència contractada. En lloc de tenir columna_1, columna_2, columna_3 i altres basuritas.

Ha de tenir un tipus, la mida és opcional:

```SQL
CREATE TABLE little_rubbish (
      keywords TEXT[],
      matrix INTEGER[3][3]
);

SELECT keywords[1] FROM little_rubbish;
```
A més hi ha [funcions de comparacio, length, fill, unnest, etc.](https://www.postgresql.org/docs/9.1/static/functions-array.html).

#### JSON
El 3/4 de la funcionalitat de Mongo es pot implementar amb PostgreSQL usant els camps JSON.
En quant a eficiència és bastant decent, l'únic que no té és map-reduce, inconsistències i fallos de seguretat.

```SQL
CREATE TABLE books ( id integer, data json );

INSERT INTO books VALUES (1,
    '{ "name": "Book the First", "author": { "first_name": "Bob", "last_name": "White" } }');

SELECT id, data->>'name' AS name FROM books;

SELECT * FROM books WHERE data->>'name' = 'Book the First';
```
A més es poden crear indexos sobre camps del JSON.


### Constraints
Els constraints serveixen per forçar certes condicions que les dades sempre han de complir.
Quan algun constraint no es compleix al fer commit la base de dades Postgres tira **un error com un piano**.
Alguns exemples són una clau primaria duplicada o una foreign key que no apunta enlloc.

#### Not Null
Impedeix valors nuls en una columna. Afegir NOT NULL és més fàcil que fer tot de comprovacions a l'aplicació o menjar-se nulls a tort i dret.
```SQL
CREATE TABLE important_numbers (  
  integer_number INTEGER NOT NULL
);
```

#### Unique
Evita valors duplicats per columna. Es pot fer compost.
```SQL
CREATE TABLE super_heroes (  
  cool_name TEXT UNIQUE
);
```

#### Check
Dóna un error si les dades no compleixen la condició indicada (una expressió que evalua a true o false).
```SQL
...
CHECK (consumption >= 0)
...
```

#### Primary Key
Indica que una columna o conjunt de columnes és un identificador únic de fila.
La primary key sempre ha de ser UNIQUE i NOT NULL.

És súper útil tenir una clau primaria per evitar duplicació de dades, inconsistències i reduïr estrès laboral.
A l'hora de fer una taula val la pena pensar bé quina ha de ser la clau primària per tenir-ho tot el més *normalitzat* possible.

#### Foreign keys
Indica que la columna referencia valors d'una altra taula:

```SQL
CREATE TABLE teams (
    id INTEGER PRIMARY KEY,
    cool_name TEXT UNIQUE
);
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    team_id INTEGER REFERENCES teams (id)
);
```

Si algú intenta desar un valor de team_id que no existeix a la taula teams li donarà error.

També es pot configurar què passa amb els foreign key al fer delete per garantir consistencia de les dades:

```SQL
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    team_id INTEGER REFERENCES teams (id) ON DELETE CASCADE
);
```

En aquest exemple, si elimines un team de la taula teams s'esborrarien automàticament els users que el referencien.

Les opcions són:
* **RESTRICT**: Tira error si intentes borrar una fila amb referències. Obligant a borrar les referències primer.
* **CASCADE**: Elimina tots els elements que referencien la fila eliminada.
* **NO ACTION**: Es deixa la referencia a una fila que no existeix!
* **SET NULL/DEFAULT**: Modifica el valor i hi fica un NULL o el valor per defecte.

Similar a ON DELETE existeix un ON UPDATE.


### Normalització

La normalització és un mètode per evitar duplicació de dades, simplificar queries i dissenyar bases de dades ben estructurades.
La següent taula d'exemple **NO** està normalitzada:

| id | name | department | floor | supervisor |
|----|------|------------|-------|------------|
| 1 | Donald | Yuge Data | 4 | Vladimir |
| 2 | Alice | Sales | 3 | Eve |
| 3 | Bob | Sales | 3 | Eve |
| 4 | Eve | Sales | 3 | Vladimir |
| 5 | Vladimir | Management | 65 | *null* |

Aquesta taula a més de tenir informació repetida té altres problemes:

#### Insert Anomaly
No es pot afegir un nou departament fins que tenim un nou treballador:

| id | name | department | floor | supervisor |
|----|------|------------|-------|------------|
| 1 | Donald | Yuge Data | 4 | Vladimir |
| 2 | Alice | Sales | 3 | Eve |
| 3 | Bob | Sales | 3 | Eve |
| 4 | Eve | Sales | 3 | Vladimir |
| 5 | Vladimir | Management | 65 | *null* |
| - | - | **IT** | **2** | - |

#### Update Anomaly
Quan es vol actualitzar la informació de departament s'han d'actualitzar multiples files. Si es fa malament es pot liar parda.

| id | name | department | floor | supervisor |
|----|------|------------|-------|------------|
| 1 | Donald | Yuge Data | 4 | Vladimir |
| 2 | Alice | Sales | **5** | Eve |
| 3 | Bob | Sales | 3 | Eve |
| 4 | Eve | Sales | 3 | Vladimir |
| 5 | Vladimir | Management | 65 | *null* |

#### Deletion Anomaly
Esborrant un usuari podem abolir un departament.

| id | name | department | floor | supervisor |
|----|------|------------|-------|------------|
| ~~1.~~ | ~~Donald~~ | ~~Yuge Data~~ | ~~4.~~ | ~~Vladimir~~ |
| 2 | Alice | Sales | 3 | Eve |
| 3 | Bob | Sales | 3 | Eve |
| 4 | Eve | Sales | 3 | Vladimir |
| 5 | Vladimir | Management | 65 | *null* |


#### Normalitzant
Comparem amb aquesta versió:

| id | name | department | supervisor |
|----|------|------------|------------|
| 1 | Donald | 1 | 5 |
| 2 | Alice | 2 | 4 |
| 3 | Bob | 2 | 4 |
| 4 | Eve | 2 | 5 |
| 5 | Vladimir | 3 | *null* |

| id | name | floor |
|----|------|-------|
| 1 | Yuge Data | 4 |
| 2 | Sales | 3 |
| 3 | Management | 65 |

Hem necessitat separar les dades en dues taules, pero ara estan "normalitzades" i amb aquesta estructura no hi haurà cap dels probemes que hi havia abans.
En general, les bases de dades normalitzades porten a taules petites amb queries senzilles, més intuitives i eficients.

La normalització és un procés formal. Hi ha llibres i [tutorials](http://databases.about.com/od/specificproducts/a/normalization.htm) que expliquen com determinar quan una taula no està normalitzada, i quines transformacions cal fer per normalitzar-la.
És molt recomanable entendre i saber aplicar la normalització, per poder crear taules amb una estructura lògica i eficient de manera intuitiva i metòdica.


Hi ha diferents nivells de normalització.
La primera forma normal (1NF) és la més baixa i s'han de posar claus primaries, tenir tot en taules bi-dimensionals (files i columnes) i poc més.
Les segona i tercera treuen duplicats i afegeixen claus foranes.
A partir de la quarta que us ho expliqui [la Wikipedia](https://en.wikipedia.org/wiki/Database_normalization#List_of_Normal_Forms).

En casos molt concrets pot ser més eficient tenir una taula no normalitzada per evitar algun join molt lent, però no és l'habitual.
És millor dissenyar taules normalitzades i desnormalitzar quan sigui necessari i sabem perquè ho estem fent.



## Queries

### Tipus de join
Explicar com van i perquè els FULL OUTER JOIN els carrega el diable.

Taula A:

| id | name |
|----|------|
| 1 | Pirate |
| 2 | Monkey |
| 3 | Ninja |
| 4 | Spaghetti |

Taula B:

| id | name |
|----|------|
| 1 | Rutabaga |
| 2 | Pirate |
| 3 | Darth Vader |
| 4 | Ninja |

#### Inner Join
```SQL
SELECT * FROM TableA
INNER JOIN TableB
ON TableA.name = TableB.name
```
| id | name | id | name |
|----|------|----|------|
| 1 | Pirate | 2 | Pirate |
| 3 | Ninja | 4 | Ninja |

![Inner Join](http://i.imgur.com/q01DA3z.png)

#### Left Outer Join
```SQL
SELECT * FROM TableA
LEFT OUTER JOIN TableB
ON TableA.name = TableB.name
```
| id | name | id | name |
|----|------|----|------|
| 1 | Pirate | 2 | Pirate |
| 2 | Monkey | *null* | *null* |
| 3 | Ninja | 4 | Ninja |
| 4 | Spaghetti | *null* | *null* |

![Left Outer Join](http://i.imgur.com/K3asUr0.png)

```SQL
SELECT * FROM TableA
LEFT OUTER JOIN TableB
ON TableA.name = TableB.name
WHERE TableB.id IS null
```
| id | name | id | name |
|----|------|----|------|
| 2 | Monkey | *null* | *null* |
| 4 | Spaghetti | *null* | *null* |

![Left Outer Join 2](http://i.imgur.com/MKfssy1.png)

#### Full Outer Join
```SQL
SELECT * FROM TableA
FULL OUTER JOIN TableB
ON TableA.name = TableB.name
```
| id | name | id | name |
|----|------|----|------|
| 1 | Pirate | 2 | Pirate |
| 2 | Monkey | *null* | *null* |
| 3 | Ninja | 4 | Ninja |
| 4 | Spaghetti | *null* | *null* |
| *null* | *null* | 1 | Rutabaga |
| *null* | *null* | 3 | Darth Vader |

![Full Outer Join](http://i.imgur.com/85tgWpY.png)

```SQL
SELECT * FROM TableA
FULL OUTER JOIN TableB
ON TableA.name = TableB.name
WHERE TableA.id IS null
OR TableB.id IS null
```
| id | name | id | name |
|----|------|----|------|
| 2 | Monkey | *null* | *null* |
| 4 | Spaghetti | *null* | *null* |
| *null* | *null* | 1 | Rutabaga |
| *null* | *null* | 3 | Darth Vader |

![Full Outer Join is null](http://i.imgur.com/s6deDYN.png)

#### Cross Join
```SQL
SELECT * FROM TableA
CROSS JOIN TableB
```
| id | name | id | name |
|----|------|----|------|
| 1 | Pirate | 1 | Rutabaga |
| 1 | Pirate | 2 | Pirate |
| 1 | Pirate | 3 | Darth Vader |
| 1 | Pirate | 4 | Ninja |
| 2 | Monkey | 1 | Rutabaga |
| 2 | Monkey | 2 | Pirate |
| 2 | Monkey | 3 | Darth Vader |
| 2 | Monkey | 4 | Ninja |
| 3 | Ninja | 1 | Rutabaga |
| 3 | Ninja | 2 | Pirate |
| 3 | Ninja | 3 | Darth Vader |
| 3 | Ninja | 4 | Ninja |
| 4 | Spaghetti | 1 | Rutabaga |
| 4 | Spaghetti | 2 | Pirate |
| 4 | Spaghetti | 3 | Darth Vader |
| 4 | Spaghetti | 4 | Ninja |

#### Tots els joins
![A drawing showing the joins](http://i.imgur.com/daJ7tX1.png)


### Indexos
```SQL
CREATE UNIQUE INDEX cups ON consumptions(cups, month);
```
* Els indexos serveixen per trobar ràpidament un subconjunt de les files d'una taula.
  * Per exemple quan la columna del index surt a la clàusula WHERE o quan es fa un JOIN.
* Per crear un index s'ha d'indicar la columna o columnes sobre les que es vol fer l'index.
* En cas de fer un index sobre multiples columnes, l'ordre és important.

#### B-Tree
* B-Tree és l'estructura de dades que fan servir els indexos per defecte.
* Permet fer cerques, insercions i eliminacions en O(log n).
* Internament guarda els nodes ordenats, amb el que les queries que facin servir l'index poden fer un ORDER BY eficient.

[Visualització de com funciona el B-Tree](https://www.cs.usfca.edu/~galles/visualization/BTree.html)

#### Altres estructures de dades

* Hash index
* Generalized Inverted Index (GIN)
* Generalized Search Tree (GiST)

#### Index parcial
```SQL
CREATE INDEX clients ON ps(atr) WHERE ekon_status IS NOT NULL;
```
#### Index sobre una expressió
```SQL
CREATE INDEX consumption_year ON EXTRACT(YEAR FROM datetime);
```


### Planificador de queries

* SQL és un llenguatge declaratiu, una query descriu quines files s'han de retornar i el query planner decideiex *com* fer la query.
* Postgres calcula estadístiques de les taules, el tipus de dades que es desen, freqüencia, etc.
* El query planner fa servir aquesta informació per decidir si fer servir un index o no, fer una taula temporal...

### Explain

Explain és una comanda que dona informació sobre com es fa una query i és molt útil per optimitzar.
Totes les bases de dades SQL ho tenen.
```SQL
EXPLAIN SELECT * FROM user;
```
Postgres mostra el resultat en text. Hi ha eines que mostren el query plan de manera gràfica.
```
Seq Scan on users  (cost=0.00..5.24 rows=234 width=41)
```
* 'Seq Scan' indica que la query recorre tota la taula.
* Cost son el cost inicial i l'estimat total recorrent tota la taula.
* Rows el nombre estimat de files resultants.
* Width la mida esperada de cada fila (en bytes).

Afegint analyze la query s'executa per tenir millor informació.
```SQL
EXPLAIN ANALYZE SELECT * FROM users;
```
Query plan:
```
Seq Scan on users (cost=0.00..5.21 rows=173 width=118)
(actual time=0.018..0.018 rows=0 loops=1)
Total runtime: 0.020 ms
```
Ull amb fer EXPLAIN ANALAZY DELETE FROM nomdelataula:
```SQL
BEGIN;
EXPLAIN ANALYZE ...;
ROLLBACK;
```

#### Scan

Scan és la forma en que es recorre la taula:

* **Sequential Scan**: Es recorre tota la taula fila a fila.
* **Index Scan**: Es recorre l'index i s'agafen de la taula les files rellevants.
* **Index Only Scan**: Es recorre l'index i no s'accedeix a la taula.
* **Bitmap Index Scan**: Es recorre l'index i es crea un bitmap per accedir a la taula en ordre.

```SQL
EXPLAIN ANALYZE SELECT * FROM user WHERE age = 35;
```
Query plan:
```
Index Scan using iuser_age on user (cost=0.00..8.27 rows=1 width=4)
Index Cond: (age = 35)
```
#### Joins
El plan de la query també indica com es fan els joins:
* **Nested Loop With Inner Sequential Scan**: Per cada element de la primera taula fa un seq scan de la segona.
* **Nested Loop With Inner Index Scan**: Per cada element de la primera taula fa el join amb l'index de la dreta.
* **Hash Join**: Crea una taula de hash sobre la taula més petita, després recorre la gran mirant si el hash coincideix.
* **Merge Join**: Ordena i fa un join (similar al merge sort).
```SQL
EXPLAIN SELECT t2.name FROM t1 JOIN t2 ON (t1.id = t2.id) WHERE t1.id = 125;
```
Query plan:
```
Nested Loop (cost=0.00..178.31 rows=320 width=28)
    -> Seq Scan on t1 (cost=0.00..152.15 rows=60 width=3)
        Filter: (id = 125::oid)
    -> Materialize (cost=0.00..37.02 rows=3 width=34)
        -> Seq Scan on t2 (cost=0.00..35.08 rows=3 width=34)
            Filter: (id = 125::oid)
```

Creant un index la cosa canvia:
```
Nested Loop (cost=0.00..18.65 rows=1 width=278)
-> Index Scan using it1_id on t1 (cost=0.00..9.32 rows=1 width=4)
Index Cond: (id = 125::oid)
-> Index Scan using it2_id on t2 (cost=0.00..9.32 rows=1 width=286)
Index Cond: (t2.id = 125::oid)
```

![Query Plan Gràfic](http://i.imgur.com/BreugDx.png)


## Referències

Transaccions i ACID:
* https://vladmihalcea.com/2014/01/05/a-beginners-guide-to-acid-and-database-transactions/

Normalització:
* https://www.essentialsql.com/get-ready-to-learn-sql-database-normalization-explained-in-simple-english/

Joins:
* https://blog.codinghorror.com/a-visual-explanation-of-sql-joins/

Indexos:
* https://devcenter.heroku.com/articles/postgresql-indexes
* http://use-the-index-luke.com/

Explain:
* http://www.vertabelo.com/blog/technical-articles/understanding-execution-plans-in-postgresql

Exercises:
* https://pgexercises.com/