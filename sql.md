# Introduzione a SQL

- [Introduzione a SQL](#introduzione-a-sql)
    - [Creare, usare e rimuovere un database](#creare-usare-e-rimuovere-un-database)
    - [Creare e rimuovere tabelle](#creare-e-rimuovere-tabelle)
    - [Inserire righe in una tabella](#inserire-righe-in-una-tabella)
    - [Selezionare dati da una tabella](#selezionare-dati-da-una-tabella)
    - [Modificare e rimuovere righe](#modificare-e-rimuovere-righe)
    - [L'arma segreta di SQL: le join](#larma-segreta-di-sql-le-join)
    - [Relazioni](#relazioni)

SQL (Structured Query Language) è un linguaggio che permette di comunicare con un DBMS (DataBase Management System). Sebbene dovrebbe essere implementation-independent (dovrebbe essere lo stesso per tutti i DBMS) spesso ogni DBMS aggiunge le proprie modifiche. Per esempio, PostreSQL aggiunge molti tipi di dato molto utili, tipo uno apposta per gli indirizzi ip, località geografiche, testo JSON, eccetera.

Ogni query SQL è costituita da parole chiave riservate dal linguaggio (tipo `CREATE`, `SELECT` eccetera), nomi di database, tabelle e campi (spesso denotati con i backticks (\`\`) ma non per forza) e valori (nelle query che inseriscono valori tipo `INSERT INTO`).  
I commenti cominciano con due trattini: `--`.  
Ogni query SQL deve terminare con il punto e virgola: `;`.

### Creare, usare e rimuovere un database

Le istruzioni `CREATE DATABASE`, `DROP DATABASE` e `USE` sono abbastanza autoesplicative

``` 
mysql> CREATE DATABASE mammamia;
Query OK, 1 row affected (0.013 sec)

mysql> USE mammamia;
Database changed

mysql> DROP DATABASE mammamia;
Query OK, 0 rows affected (0.016 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| phpmyadmin         |
| sito_esempio       |
| test               |
+--------------------+
6 rows in set (0.004 sec)
``` 

### Creare e rimuovere tabelle

Per creare una tabella dobbiamo specificare i nomi e tipi delle colonne, la chiave primaria, eventuali constraints (vedremo più avanti), indici (vedremo più avanti).

``` 
mysql> CREATE TABLE test(
    -> id INT NOT NULL,
    -> testo VARCHAR(255),
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.052 sec)
``` 

Notare come anche il terminale mi ha lasciato andare a capo fino a che non ho messo il punto e virgola.  
Quindi abbiamo creato una tabella con chiave primaria id e un campo solo. Ma la chiave primaria dovremo specificarla ogni volta che inseriamo un campo. Come facciamo un "autoincrement" come faceva Access? Questa è una delle differenze tra i vari DBMS. Postres usa `SERIAL`, SQL Server usa `GENERATED ALWAYS AS IDENTITY`, MySQL usa `AUTO_INCREMENT`.

``` 
mysql> CREATE TABLE test(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> testo VARCHAR(255),
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.054 sec)

mysql> DESC test;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int(11)      | NO   | PRI | NULL    | auto_increment |
| testo | varchar(255) | YES  |     | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
2 rows in set (0.025 sec)
``` 

Per rimuovere una tabella il comando è semplicissimo (ricordate il nostro amico [Bobby Tables?](https://xkcd.com/327/)).

``` 
mysql> DROP TABLE test;
Query OK, 0 rows affected (0.017 sec)
``` 

### Inserire righe in una tabella

L'istruzione `INSERT INTO` permette di creare una nuova riga in una tabella e inserirci dei valori. Possiamo scegliere in quali campi inserire dei valori specificandoli nelle parentesi.

``` 
mysql> INSERT INTO TEST(testo) VALUES ('Questa è una riga che inserisco da SQL');
Query OK, 1 row affected (0.006 sec)

mysql> SELECT * FROM test;
+----+----------------------------------------+
| id | testo                                  |
+----+----------------------------------------+
|  1 | Questa è una riga che inserisco da SQL |
+----+----------------------------------------+
1 row in set (0.001 sec)
``` 

Vedremo come modificare e rimuovere righe più avanti, dopo aver visto l'istruzione `WHERE`

### Selezionare dati da una tabella

L'istruzione `SELECT` ci mostra il contenuto di una tabella. Possiamo scegliere quali campi vedere elencandoli dopo `SELECT`

``` 
mysql> SELECT * FROM test;
+----+----------------------------------------+
| id | testo                                  |
+----+----------------------------------------+
|  1 | Questa è una riga che inserisco da SQL |
+----+----------------------------------------+
1 row in set (0.001 sec)

MariaDB [test]> SELECT testo FROM test;
+----------------------------------------+
| testo                                  |
+----------------------------------------+
| Questa è una riga che inserisco da SQL |
+----------------------------------------+
1 row in set (0.001 sec)
``` 

Possiamo impostare dei criteri di selezione con l'istruzione `WHERE`. In questo esempio creeremo una tabella con due campi "nome" e "età", poi chiederemo a MySQL, inseriremo due persone, e chiederemo a MySQL di restituirci solo le persone maggiorenni.  

``` 
mysql> CREATE TABLE test_due(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> nome VARCHAR(255) NOT NULL,
    -> età INT NOT NULL,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.024 sec)

mysql> INSERT INTO test_due(nome, età) VALUES('Giovanni', 17), ('Lorenzo', 18);
Query OK, 2 rows affected (0.007 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM test_due WHERE età >= 18;
+----+---------+-----+
| id | nome    | età |
+----+---------+-----+
|  2 | Lorenzo |  18 |
+----+---------+-----+
1 row in set (0.000 sec)
``` 

> Notare come il nome del campo "età" contenga la a accentata. Questa è generalmente considerata una cattiva pratica. È consigliato usare solo caratteri ASCII in questo genere di software a base testuale. Se un giorno dovessimo risolvere qualche problema collegandoci da terminale remoto potremmo non aver accesso alla tastiera italiana.

### Modificare e rimuovere righe

Mettiamo che il nostro Giovanni abbia compiuto 18 anni. Dobbiamo aggiornare la sua riga e impostare la sua età a 18. La sintassi per questa operazione è molto simile al `SELECT qualcosa qualcosa WHERE`.

``` 
mysql> UPDATE test_due SET età=18 WHERE nome='Giovanni';
Query OK, 1 row affected (0.004 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT * FROM test_due;
+----+----------+-----+
| id | nome     | età |
+----+----------+-----+
|  1 | Giovanni |  18 |
|  2 | Lorenzo  |  18 |
+----+----------+-----+
2 rows in set (0.000 sec)
``` 

Per rimuovere una riga la sintassi è quasi identica

``` 
mysql> DELETE FROM test_due WHERE nome='Giovanni';
Query OK, 1 row affected (0.005 sec)

mysql> SELECT * FROM test_due;
+----+---------+-----+
| id | nome    | età |
+----+---------+-----+
|  2 | Lorenzo |  18 |
+----+---------+-----+
1 row in set (0.001 sec)
``` 

### L'arma segreta di SQL: le join

Mettiamo il caso di un social network molto molto semplice. Questo social ha degli utenti, e ogni utente può pubblicare dei post. Come abbiamo visto in informatica, il metodo più semplice per rappresentare questo social sarebbe con due tabelle: una per gli utenti e una per i post.

``` 
mysql> CREATE TABLE utenti(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> nome VARCHAR(255) NOT NULL,
    -> email VARCHAR(255) NOT NULL,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.036 sec)

mysql> CREATE TABLE post(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> autore INT NOT NULL,
    -> contenuto VARCHAR(1024) NOT NULL,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.028 sec)
``` 

Mi sono preso la libertà di inserire alcuni post e utenti manualmente

``` 
mysql> SELECT * FROM utenti;
+----+----------+--------------------------+
| id | nome     | email                    |
+----+----------+--------------------------+
|  1 | Lorenzo  | bottilorenzobg@gmail.com |
|  2 | Giovanni | canemagico@botti.com     |
+----+----------+--------------------------+
2 rows in set (0.000 sec)

mysql> SELECT * FROM post;
+----+--------+--------------------------------------------------------------+---------------------+
| id | autore | contenuto                                                    | data_pubblicazione  |
+----+--------+--------------------------------------------------------------+---------------------+
|  1 |      1 | Uelà, sono Lorenzo e mi sono appena iscritto                 | 2021-01-27 12:01:45 |
|  2 |      2 | Io invece sono Giovanni e anch'io mi sono appena iscritto!   | 2021-01-27 14:01:45 |
|  3 |      1 | Mi sono iscritto ieri e sono un pò stanchino, mi sa che esco | 2021-01-28 12:02:47 |
+----+--------+--------------------------------------------------------------+---------------------+
3 rows in set (0.000 sec)
``` 

Bello e tutto, ma io vorrei vedere i post e il loro autore nella stessa tabella, e se volessi vederli insieme?  
La soluzione a questo problema è una `JOIN`: un'istruzione per unire dati da due tabelle in base a una certa condizione.  
Mettiamo che io voglia vedere l'autore del primo post. Dovrei leggere dalla tabella `post` il campo `autore`, poi cercare nella tabella `utenti` la riga che ha come `id` l'`autore` del post. Ok vabè senti se stai leggendo questa parte non l'ho ancora finita di scrivere, fai finta di niente, mi serve un bel voto

``` 
mysql> SELECT utenti.nome, post.contenuto, post.data_pubblicazione FROM post JOIN utenti ON post.autore = utenti.id ORDER BY post.data_pubblicazione DESC;
+----------+--------------------------------------------------------------+---------------------+
| nome     | contenuto                                                    | data_pubblicazione  |
+----------+--------------------------------------------------------------+---------------------+
| Lorenzo  | Mi sono iscritto ieri e sono un pò stanchino, mi sa che esco | 2021-01-28 12:02:47 |
| Giovanni | Io invece sono Giovanni e anch'io mi sono appena iscritto!   | 2021-01-27 14:01:45 |
| Lorenzo  | Uelà, sono Lorenzo e mi sono appena iscritto                 | 2021-01-27 12:01:45 |
+----------+--------------------------------------------------------------+---------------------+
3 rows in set (0.000 sec)
``` 

### Relazioni
Nella tabella `post` il campo `autore` esiste solo ed esclusivamente per riferirsi al campo `id` della tabella `utenti`. Ma cosa succede se elimino un utente dopo che ha già postato qualcosa? Rimarranno delle righe "fantasma" nella tabella `post` che non posso trovare con la `JOIN` (a meno che non uso l'`OUTER JOIN` ma è roba più avanzata). Una possibile soluzione può essere far gestire questa relazione al DBMS, e fargli eliminare tutti i post di un utente quando questi viene eliminato. Per comunicare questo ordine a MySQL usiamo un `CONTRAINT`, in questo caso, una foreign key.

```
mysql> CREATE TABLE utenti(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> nome VARCHAR(255) NOT NULL,
    -> email VARCHAR(255) NOT NULL,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.036 sec)

mysql> CREATE TABLE post(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> autore INT NOT NULL,
    -> contenuto VARCHAR(1024) NOT NULL,
    -> PRIMARY KEY(id),
    -> FOREIGN KEY (autore) REFERENCES utenti (id) ON DELETE CASCADE ON UPDATE CASCADE
    -> );
Query OK, 0 rows affected (0.028 sec)
```

Se abbiamo già creato una tabella e vogliamo aggiungere un constraint dopo possiamo usare il comando `ALTER TABLE ADD CONSTRAINT`

```
mysql> ALTER TABLE post ADD CONSTRAINT fk_autore FOREIGN KEY(autore) REFERENCES utenti(id) ON DELETE CASCADE ON UPDATE CASCADE;
Query OK, 0 rows affected (0.083 sec)
Records: 0  Duplicates: 0  Warnings: 0
```


Adesso oltre alla NASA puoi anche hackerare la CIA che ne so, buona fortuna nel tuo percorso da criminale
