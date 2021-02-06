# PHP: Introduzione a MySQL e PDO

- [PHP: Introduzione a MySQL e PDO](#php-introduzione-a-mysql-e-pdo)
  - [Connettersi al database](#connettersi-al-database)
  - [Eseguire una query](#eseguire-una-query)
  - [Eseguire una query che restituisce una tabella](#eseguire-una-query-che-restituisce-una-tabella)
  - [Query con parametri](#query-con-parametri)

PDO (PHP Data Objects) è lo standard de facto per accedere a un DBMS relazionale da PHP. PDO definisce un'interfaccia comune per comunicare con vari DBMS come MySQL, PostgreSQL, SQLite e molti altri

## Connettersi al database
Spesso i DBMS sono servizi separati dal nostro programma, perciò per farne uso dal nostro codice dobbiamo effettivamente connetterci a MySQL come se fosse un servizio web.

```
try {
    $hostname = "localhost";
    $dbname = "nome_database";
    $user = "nome_utente";
    $pass = "password";
    // Apri una connessione
    $db = new PDO ("mysql:host=$hostname;dbname=$dbname", $user, $pass);
} catch (PDOException $e) {
    // Controllo per errori
    echo "Errore: " . $e->getMessage();
    die();
}
```   
## Eseguire una query

Se la query che vogliamo eseguire non restituisce una tabella, il metodo query() ritorna un semplice booleano, `true` in caso di successo e  `false` in caso di errore

```  
$sql = "INSERT INTO Utenti (username, password, email)
VALUES ('Giovanni', '1234', 'giovanni@example.com')";

if ($conn->query($sql) === TRUE) {
    echo "Record scritto correttamente";
} else {
    echo "Errore: " . $sql . " " . $conn->error;
}

$conn->close();
```

## Eseguire una query che restituisce una tabella

Una query che restituisce una tabella (tipo "SELECT") invece restituisce un oggetto PDOStatement. Questo oggetto ha una funzione fetch() che ogni volta che viene chiamata restituisce una riga diversa. Questa riga viene rappresentata come un array con indici stringa (in linguaggi che hanno senso si chiamano "dizionari"). La riga `while($riga = $risultato->fetch())` fa in modo che il contenuto di quel ciclo while venga eseguito per ogni riga.

```
$sql = "SELECT * FROM Utenti";
$risultato = $conn->query($sql);

if ($risultato->numRows() > 0) {
    // Stampa i dati di ogni riga
    while($riga = $risultato->fetch()) {
        echo "id: " . $riga["id"]. " - Name: " . $riga["username"]. " " . $riga["email"]. "\n";
    }
} else {
    echo "0 risultati";
}
```
Per esempio, ecco cosa succede quando eseguo la stessa query direttamente nel DBMS
```
mysql> SELECT * FROM utenti;
+-----------+----------+------------+------------------+-----------------------------------------+
| id_utente | username | password   | nome_pubblico    | biografia                               |
+-----------+----------+------------+------------------+-----------------------------------------+
|         1 | mathik   | burger     | Lorenzo Botti    | Ciao, sono Lorenzo Botti                |
|         2 | test1    | canemagico | Giovanni Giorgio | Questo utente esiste davvero ;)         |
|         3 | test2    | bbb        | Orazio           | carpe diem quam minimum credero postero |
|         4 | marcello | bbb        | Jimi Hendrix     | Dove ho messo la chitarra?              |
|         7 | claire   | bbb        | Claire           | fan di questo sito                      |
+-----------+----------+------------+------------------+-----------------------------------------+
```

## Query con parametri
L'SQL Injection è una forma di attacco informatico che consiste nel inserire caratteri interpretabili da MySQL in una stringa che si manda al server, in modo tale da ingannare il server a eseguire codice mandato dall'utente. Possiamo difenderci da questo attacco sanificando l'input che inseriamo nel database o usando **query parametrizzate** Questo famoso fumetto di [xkcd](https://xkcd.com/ "Xkcd site") rappresenta più o meno l'attacco.

![Bobby tables](https://imgs.xkcd.com/comics/exploits_of_a_mom.png "Bobby tables")

```
// Sostuisco le variabili nella query con dei placeholders
// Così da trattare i valori inviati come dati, non come istruzioni
// Si passa da questo (query con variabili - invio istruzioni):
$sql = "SELECT * FROM users WHERE email = '$email' AND status='$status'";

// A questo (query con placeholders - invio dati):
$sql = 'SELECT * FROM users WHERE email = ? AND status=?';

// In sostanza:
// Preparo la query
$stmt = $pdo->prepare($sql);

// La eseguo passando i parmetri
$stmt->execute([$email, $status]);
```
Bravo! Adesso sei un esperto di MySQL, puoi hackerare la NASA, eccetera. Clicca qui per imparare [l'SQL](Link da inserire "Pagina sql") adesso!
