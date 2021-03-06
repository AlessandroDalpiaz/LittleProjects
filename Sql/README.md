# Normalizzazione relazionale
La normalizzazione conclude un database.
La chiave candidata é un insieme di uno o più attributi che possono diventare chiave primaria, ogni chiave candidata individua in maniera univoca ogni record della mia tabella, e determinano ogni attributo. La chiave primaria è la più piccola tra le chiave candidate. La dipendenza funzionale quando il valore di un insieme degli attributi A determina un singolo valore degli attributi di B. Dipendenza transitiva quando A determina B e B determina C, quindi A determina transitivamente C.

### Comandi SQL
Seleziona la tabella nomi con almeno 6 lettere
```
SELECT nome AS Nomi
FROM automobilista
WHERE nome LIKE '_%_____'
```
Seleziona i nomi che abitano a Venezia o Trivignano
```
SELECT nome AS Nomi
FROM automobilista
WHERE Citta IN ('Venezia','Trivignano')
```
SQL JOIN 
```
SELECT Orders.OrderID, Customers.CustomerName, Orders.OrderDate
FROM Orders
INNER JOIN Customers ON Orders.CustomerID=Customers.CustomerID;
```
SQL LEFT JOIN e SQL RIGHT JOIN
```
non molto utile
```
JOIN con WHERE
```
SELECT automobilista.Nome, automobilista.Codice_Fiscale, auto.Targa, infrazioni.Data
FROM auto, automobilista, infrazioni
WHERE automobilista.Codice_Fiscale = auto.FK_Codice_Fiscale AND
infrazioni.FK_Targa = auto.Targa
```
JOIN di 4 tabelle 
```
SELECT automobilista.Nome, automobilista.Codice_Fiscale, auto.Targa, infrazioni.Data, agenti.Matricola
FROM auto, automobilista, infrazioni, agenti
WHERE automobilista.Codice_Fiscale = auto.FK_Codice_Fiscale AND
infrazioni.FK_Targa = auto.Targa AND
infrazioni.FK_Matricola = agenti.Matricola
```
SELECT UNION

GROUP BY

Mostra il numero di multe che ogni agente ha fatto con alias Numero
```
SELECT agenti.Nome, COUNT(infrazioni.ID_Infrazione) AS Numero_Di_Infrazioni
FROM infrazioni
JOIN agenti ON infrazioni.FK_Matricola = agenti.Matricola
GROUP BY agenti.Matricola
ORDER BY Numero_Di_Infrazioni
```
Seleziona nome, matricola e somma del numero di infrazioni e dell’importo di infrazioni e le ordina in modo decrescente per somma dell’importo delle infrazioni
```
SELECT agenti.Matricola, agenti.Nome, SUM(infrazioni.Importo) AS Importo_totale_infrazioni, COUNT(*) AS Numero_di_infrazioni
FROM infrazioni, agenti
WHERE infrazioni.FK_Matricola = agenti.Matricola
GROUP BY agenti.Matricola
ORDER BY Importo_totale_infrazioni DESC
```
Numero di macchine che ogni automobilista possiede
```
SELECT automobilista.Codice_Fiscale, automobilista.Nome, COUNT(*) AS Somma_delle_macchine
FROM automobilista, auto
WHERE automobilista.Codice_Fiscale = auto.FK_Codice_Fiscale
GROUP BY automobilista.Codice_Fiscale
ORDER BY automobilista.Codice_Fiscale DESC
```
Numero di tipi di infrazioni che hanno commesso solo le macchine Fiat
```
SELECT infrazioni.Tipo_Di_Infrazione, COUNT(*) AS Numero_di_infrazioni
FROM auto, infrazioni
WHERE infrazioni.FK_Targa = auto.Targa AND auto.Marca = 'Fiat'
GROUP BY infrazioni.Tipo_Di_Infrazione
```
SQL HAVING Si usa per mettere una condizione, mentre GROUP BY li seleziona tutti:

Nome e indirizzo di automobilisti cui sono contestate almeno 1 infrazione per “Divieto di sosta”
```
SELECT automobilista.Nome, infrazioni.Tipo_Di_Infrazione, COUNT(*) AS Numero_di_infrazioni
FROM infrazioni, automobilista, auto
WHERE infrazioni.FK_Targa = auto.Targa AND automobilista.Codice_Fiscale = auto.FK_Codice_Fiscale AND infrazioni.Tipo_Di_Infrazione = 'Parcheggio in divieto di sosta'
GROUP BY automobilista.Nome HAVING COUNT(*) >= 1
```
Per ogni prestito il valore del prestito e i dati identificativi dei clienti che lo hanno stipulato
```
SELECT prestito.importo, clienti.nome
FROM prestito, clienti, cliente_prestito
WHERE cliente_prestito.FK_ID_cliente=clienti.ID_cliente AND prestito.ID_prestito = cliente_prestito.FK_ID_prestito
```
Elenco di tutti i clienti che hanno almeno un deposito e almeno un prestito
```
SELECT DISTINCT clienti.nome
FROM clienti, prestito, conto, clienti_cc, cliente_prestito, filiale
WHERE prestito.ID_prestito = cliente_prestito.FK_ID_prestito AND clienti.ID_cliente = clienti_cc.FK_ID_cliente AND clienti.ID_cliente = cliente_prestito.FK_ID_cliente
```
Elenco di tutti i clienti titolari di almeno un deposito ma non di un prestito
```
SELECT DISTINCT clienti.nome
FROM clienti WHERE clienti.nome NOT IN (SELECT clienti.nome FROM clienti, prestito, cliente_prestito
WHERE prestito.ID_prestito = cliente_prestito.FK_ID_prestito AND clienti.ID_cliente = cliente_prestito.FK_ID_cliente)
```
Per ogni filiale il numero dei titolari di conti correnti (controllare ogni cliente una sola volta)
```
SELECT filiale.ID_filiale, filiale.citta, COUNT(*) AS Numero_di_conti
FROM filiale, conto
WHERE filiale.ID_filiale = conto.FK_ID_filiale
GROUP BY filiale.ID_filiale
```
Elenco di tutte le filiali che hanno un patrimonio maggiore del più piccolo capitale delle filiali di Venezia
```
SELECT filiale.indirizzo_filiare, filiale.patrimonio, filiale.ID_filiale
FROM filiale
WHERE filiale.patrimonio > (SELECT MIN(filiale.patrimonio) FROM filiale WHERE filiale.citta = 'venezia')
```
Nome della filiale o delle filiali con il saldo medio più alto - da sistemare
```
SELECT filiale.ID_filiale, MAX(media) 
FROM filiale, (SELECT filiale.ID_filiale AVG(saldo) AS 'media'
FROM conto, filiale
WHERE conto.FK_ID_filiale = filiale.ID_filiale
GROUP BY conto.FK_ID_filiale) as T
```
Elenco di clienti che hanno un deposito presso tutte le filiali di Venezia
```
SELECT clienti.nome, filiale.citta, filiale.indirizzo_filiare
FROM clienti_cc, conto, filiale, clienti
WHERE clienti_cc.FK_ID_cliente=clienti.ID_cliente AND clienti_cc.FK_ID_conto=conto.ID_conto AND conto.FK_ID_filiale=filiale.ID_filiale AND filiale.citta='venezia'
```
Patrimonio medio delle filiali
```
SELECT filiale.indirizzo_filiare, AVG(filiale.patrimonio) AS media, SUM(conto.saldo) AS somma
FROM filiale, conto
WHERE filiale.ID_filiale=conto.FK_ID_filiale
GROUP BY filiale.indirizzo_filiare HAVING AVG(conto.saldo)>10000
```
Nome degli attori che hanno recitato almeno due volte un film di genere horror
```
SELECT genere.Descrizione, attori.Nominativo
FROM genere, film, attori, recita_in
WHERE genere.Id_Genere=film.Id_Genere AND attori.Id_Attore=recita_in.Id_Attore AND film.Id_Film=recita_in.Id_Film AND genere.Descrizione='Horror'
GROUP BY attori.Nominativo HAVING COUNT(genere.Descrizione)>=1
```
Quenti film ci sono per genere
```
SELECT COUNT(film.Titolo),genere.Descrizione
FROM genere,film
WHERE genere.Id_Genere=film.Id_Genere
GROUP BY genere.Id_Genere
```
Quenti film per genere sono maggiori di 2
```
SELECT COUNT(film.Titolo),genere.Descrizione
FROM genere,film
WHERE genere.Id_Genere=film.Id_Genere
GROUP BY genere.Id_Genere HAVING COUNT(film.Titolo)>=2
```
Titolo e numero di premi vinti da ogni film premiato nell’anno 2003
```
SELECT film.Titolo, COUNT(*) AS Numero_Premi
FROM film, premi, ha_vinto
WHERE film.Id_Film=ha_vinto.Id_Film AND premi.Id_Premio=ha_vinto.Id_Premio AND ha_vinto.Anno='2001'
GROUP BY film.Titolo
```
Film di genere Animazione, quello (solo 1) che ha minor durata
```
SELECT film.Titolo, genere.Descrizione, MIN(film.Durata)
FROM film, genere
WHERE genere.Descrizione='Animazione' AND genere.Id_Genere=film.Id_Genere
```
Film di genere Animazione, quelli (anche più di uno) che hanno minor durata
```
SELECT film.Titolo, genere.Descrizione
FROM film, genere
WHERE genere.Descrizione='Animazione' 
AND genere.Id_Genere=film.Id_Genere
AND film.Durata=(SELECT MIN(film.Durata)
FROM film, genere
WHERE genere.Id_Genere=film.Id_Genere
AND genere.Descrizione='Animazione')
```
Film di un attore tutti i film ordinati per anno
```
SELECT attori.Nominativo, film.Titolo, film.Anno
FROM film,recita_in,attori
WHERE attori.Id_Attore=recita_in.Id_Attore AND film.Id_Film=recita_in.Id_Film AND attori.Id_Attore='6'
GROUP BY film.Anno
```
### Gianvitto Bono

A
```
SELECT atleti.nazione, atleti.nome, atleti.cognome
FROM atleti
WHERE atleti.nazione = 'gbr' OR
atleti.nazione = 'usa'
ORDER BY atleti.nazione, atleti.cognome, atleti.nome DESC
```
B
```	
SELECT atleti.nome, atleti.cognome
FROM atleti, gareatleti
WHERE gareatleti.id_atleta = atleti.id_atleta AND
atleti.nazione = 'ita' AND
gareatleti.anno LIKE '2004' 
```
C
```	  
SELECT atleti.nazione
FROM atleti, gareatleti, gare
WHERE gareatleti.id_gara = gare.id_gara AND
gareatleti.id_atleta = atleti.id_atleta AND
gareatleti.anno = '2008' AND
gareatleti.posizione >= 1 AND gareatleti.posizione <= 3
GROUP BY atleti.nazione
HAVING COUNT(*) >= 1
```
D
```
SELECT atleti.nazione, gareatleti.posizione, COUNT(*) AS 'Numero_medaglie'
FROM atleti, gareatleti, gare
WHERE gareatleti.id_gara = gare.id_gara AND
gareatleti.id_atleta = atleti.id_atleta AND
gareatleti.anno = '2004' AND
gareatleti.posizione >= 1 AND gareatleti.posizione <= 3
GROUP BY atleti.nazione, gareatleti.posizione 
ORDER BY atleti.nazione, gareatleti.posizione ASC
```
E
```
SELECT atleti.nome, atleti.cognome, atleti.nazione
FROM atleti, gareatleti, gare
WHERE gareatleti.id_gara = gare.id_gara AND
gareatleti.id_atleta = atleti.id_atleta AND
gareatleti.posizione = '1' AND
gare.genere = 'f'
```
F
```
SELECT atleti.nazione, COUNT(*) AS 'Medaglie_vinte'
FROM atleti, gareatleti
WHERE gareatleti.id_atleta = atleti.id_atleta AND
gareatleti.posizione = '1'
GROUP BY atleti.nazione
HAVING Medaglie_Vinte = (SELECT MAX(Medaglie_Vinte)
FROM (SELECT COUNT(*) AS 'Medaglie_vinte'
FROM atleti, gareatleti
WHERE gareatleti.id_atleta = atleti.id_atleta AND
gareatleti.posizione = '1'
GROUP BY atleti.nazione
)AS T)
```
G
```
SELECT gare.genere, COUNT(*) AS 'N_Ori'
FROM gare, gareatleti, specialita, atleti
WHERE gareatleti.id_gara = gare.id_gara AND
gareatleti.id_atleta = atleti.id_atleta AND
gare.id_specialita = specialita.id_specialita AND
specialita.descrizione = 'nuoto' AND
atleti.nazione = 'usa'
GROUP BY gare.genere
ORDER BY N_Ori DESC
```
H
```
SELECT atleti.nazione, COUNT(*) AS 'N_Ori'
FROM atleti, gare, gareatleti, specialita
WHERE gareatleti.id_gara = gare.id_gara AND	
gareatleti.id_atleta = atleti.id_atleta AND
gare.id_specialita = specialita.id_specialita AND
gareatleti.posizione = '1' AND
specialita.descrizione = 'scherma' 
GROUP BY atleti.nazione
HAVING N_Ori >= 3
ORDER BY N_Ori DESC
```
I
```
SELECT atleti.nazione, COUNT(*) AS 'N_Ori'
FROM atleti, gare, gareatleti
WHERE gareatleti.id_gara = gare.id_gara AND
gareatleti.id_atleta = atleti.id_atleta AND
gare.descrizione = '200m piani' AND
gareatleti.posizione = '1' AND
atleti.id_atleta NOT IN (SELECT atleti.id_atleta
FROM atleti, gare, gareatleti
WHERE gareatleti.id_gara = gare.id_gara AND
gareatleti.id_atleta = atleti.id_atleta AND
gare.descrizione = '100m piani' AND
gareatleti.posizione = '1')
GROUP BY atleti.nazione
ORDER BY N_Ori DESC
```
### Sartorio
A
```
SELECT atleti.nazione, atleti.cognome, atleti.nome
FROM atleti
WHERE atleti.nazione='gbr' OR atleti.nazione='usa'
ORDER BY atleti.nazione ASC
```
B
```
SELECT gareatleti.anno, atleti.cognome, atleti.nome
FROM atleti, gareatleti
WHERE atleti.id_atleta=gareatleti.id_atleta AND
gareatleti.anno='2004'AND
atleti.nazione='ita'
ORDER BY atleti.cognome ASC
```
C
```
SELECT atleti.nazione
FROM atleti, gareatleti
WHERE atleti.id_atleta=gareatleti.id_atleta AND
gareatleti.anno='2008' AND
gareatleti.posizione<=3
GROUP BY atleti.nazione
```
D
```
SELECT atleti.nazione, gareatleti.posizione
FROM atleti, gareatleti
WHERE atleti.id_atleta=gareatleti.id_atleta AND
gareatleti.anno='2004' AND
gareatleti.posizione<=3
```
E
```
SELECT atleti.cognome, atleti.nome
FROM atleti, gareatleti, gare
WHERE atleti.id_atleta=gareatleti.id_atleta AND
gareatleti.id_gara=gare.id_gara AND
gareatleti.posizione='1' AND
gare.genere='f'
GROUP BY atleti.cognome
```
F

G

H
```
SELECT atleti.nazione
FROM atleti, gareatleti, gare, specialita
WHERE atleti.id_atleta=gareatleti.id_atleta AND
gareatleti.id_gara=gare.id_gara AND
gare.id_specialita=specialita.id_specialita AND
specialita.descrizione='scherma'
GROUP BY atleti.nazione HAVING COUNT(gareatleti.posizione=1)
```
I
```
SELECT atleti.nazione, specialita.descrizione, gare.descrizione
FROM atleti, gareatleti, gare, specialita
WHERE atleti.id_atleta=gareatleti.id_atleta AND
gareatleti.id_gara=gare.id_gara AND
gare.id_specialita=specialita.id_specialita AND
specialita.descrizione='atletica leggera'AND
gare.descrizione='200m piani'AND
gareatleti.posizione=1 AND NOT
atleti.nazione=(SELECT atleti.nazione
FROM atleti, gareatleti, gare, specialita
WHERE atleti.id_atleta=gareatleti.id_atleta AND
gareatleti.id_gara=gare.id_gara AND
gare.id_specialita=specialita.id_specialita AND
specialita.descrizione='atletica leggera'AND
gare.descrizione='100m piani'AND
gareatleti.posizione>=3)
GROUP BY atleti.nazione
```