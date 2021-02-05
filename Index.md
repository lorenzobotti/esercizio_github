# PHP: Introduzione a SQL e MySQLi
## **Indice**
- Connettersi al database
- Eseguire una query
- Eseguire una query che restituisce una tabella
- Query con parametri
- Basi di SQL

MySQLi è una libreria per semplificare la connessione e l'uso di MySQL tramite PHP. Spesso arriva già installata con PHP, in XAMPP è preinstallata. Si può usare in due modi: usando un paradigna procedurale (stile C) o a oggetti (stile Java e C++)

## **Connettersi al database**
Spesso i DBMS sono servizi separati dal nostro programma, perciò per farne uso dal nostro codice dobbiamo effettivamente connetterci a MySQL come se fosse un servizio web.

Metodo PDO:
```
try {
        $hostname = "localhost";
        $dbname = "nome_database";
        $user = "nome_utente";
        $pass = "password";
        // Apri una connessione
        $db = new PDO ("mysql:host=$hostname;dbname=$dbname", $user, $pass);
    } catch (PDOException $e) { // Controllo per errori
        echo "Errore: " . $e->getMessage();
        die();
    }

## **Eseguire una query**
```     
$sql = "INSERT INTO Utenti (username, password, email)
VALUES ('Giovanni', '1234', 'giovanni@example.com')";

if ($conn->query($sql) === TRUE) {
echo "Record scritto correttamente";
} else {
echo "Errore: " . $sql . "
" . $conn->error;
}

$conn->close();
```

## **Eseguire una query che restituisce una tabella**

Una query che restituisce una tabella (tipo "SELECT") restituisce un oggetto mysqli_result. Questo oggetto ha una funzione fetch_assoc() che ogni volta che viene chiamata restituisce una riga diversa. Questa riga viene rappresentata come un array con indici a stringa (in linguaggi che hanno senso si chiamano "dizionari") La riga `while($riga = $risultato->fetch_assoc())` fa in modo che il contenuto di quel ciclo while venga eseguito per ogni riga.
```
$sql = "SELECT * FROM Utenti";
$risultato = $conn->query($sql);

if ($risultato->num_rows > 0) {
// Stampa i dati di ogni riga
while($riga = $risultato->fetch_assoc()) {
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
## **Query con parametri**
L'SQL Injection è una forma di attacco informatico che consiste nel inserire caratteri interpretabili da MySQL in una stringa che si manda al server, in modo tale da ingannare il server a eseguire codice mandato dall'utente. Possiamo difenderci da questo attacco sanificando l'input che inseriamo nel database o usando **query parametrizzate** Questo famoso fumetto di [xkcd](https://xkcd.com/ "Xkcd site") rappresenta più o meno l'attacco.

![Bobby tables](https://imgs.xkcd.com/comics/exploits_of_a_mom.png "Bobby tables")

```
// preparo la query
$stmt = $mysqli -> prepare("INSERT INTO Utenti (username, password, email) VALUES (?, ?, ?)");
//questi parametri sono passati per riferimento, non per valore
//ciò fa in modo che posso rifare la query senza richiamare bind_param()
//ma attenti!
$stmt -> bind_param("sss", $username, $password, $email);

// imposta i parametri
// notare che facciamo questa cosa dopo averli già passati a bind_param()
// perché sono passati per riferimento
$username = "mammamia";
$password = "marcello";
$email = "mario@example.com";

//eseguo la query
$stmt -> execute();
```
Bravo! Adesso sei un esperto di MySQL, puoi hackerare la NASA, eccetera. Clicca qui per imparare [l'SQL](Link da inserire "Pagina sql") adesso!
