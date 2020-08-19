---
title: "Designing databases"
permalink: /part5/
nav_order: 6
published: true
---

# Designing databases

Suunnittelun periaatteet
Tietokannan suunnittelussa meidän tulee päättää, mitä tauluja tietokannassa on sekä mitä sarakkeita kussakin taulussa on. Tähän on sinänsä suuri määrä mahdollisuuksia, mutta tuntemalla muutaman periaatteen ja maalaisjärjellä pärjää hyvin pitkälle.

Hyvä tavoite suunnittelussa on, että tuloksena olevaa tietokantaa on mukavaa käyttää SQL-kielen avulla. Tietokannan rakenteen tulisi olla sellainen, että pystymme hakemaan ja muuttamaan tietoa näppärästi SQL-komennoilla.

Tässä on neljä periaatetta tietokannan suunnitteluun:

Periaate 1
Tietokannan taulut ja niiden sarakkeet ovat kiinteät, ja tietokannan käyttäjä tekee muutoksia vain riveihin. Kaikki saman tyyppistä tietoa sisältävät rivit ovat samassa taulussa.

Periaate 2
Jokaisessa sarakkeessa on jokin yksittäinen tieto, kuten yksi luku tai merkkijono, mutta ei listaa tiedoista. Lista tallennetaan erilliseen tauluun niin, että jokainen alkio on oma rivinsä.

Periaate 3
Jokainen tieto on tasan yhdessä paikassa tietokannassa. Muualla tietoon viitataan rivin pääavaimen (käytännössä id-numeron) perusteella.

Periaate 4
Tietokannassa ei ole tietoa, jonka voi laskea tai päätellä tietokannan muun sisällön perusteella.

Normalisointi
Tietokantojen teoriassa esiintyy usein käsite normalisointi, jonka tavoitteena on tietokannan rakenteen parantaminen. Tämä tapahtuu muokkaamalla tietokantaa niin, että se toteuttaa tiettyjä normaalimuotoja.

Normalisointi johtaa käytännössä samaan tulokseen kuin yllä olevat periaatteet, mutta normaalimuotojen vaatimukset ovat varsin kryptiset. Jos haluat ajanvietteeksi perehtyä tietokantojen teoriaan, sinun kannattaa ottaa selvää normaalimuodoista – muuten yllä olevat periaatteet riittävät mainiosti.

Esimerkki
Tarkastellaan esimerkkinä tilannetta, jossa nettisivustolla on käyttäjiä ja jokaisella käyttäjällä on lista kavereista. Seuraavassa on huonosti suunniteltu taulu Kayttajat, joka sisältää tiedot käyttäjistä ja heidän kavereistaan:

id          tunnus      kaverit       yhteensa
----------  ----------  ------------  ----------
1           uolevi      maija,liisa   2
2           maija       aapeli        1
3           liisa                     0
4           aapeli      uolevi,maija  2
Taulussa on ideana, että sarakkeessa kaverit on lista kavereista merkkijonona, jossa kaverien tunnukset on erotettu pilkuilla. Lisäksi sarakkeessa yhteensa kerrotaan, montako kaveria käyttäjällä on yhteensä.

Tietokanta rikkoo tällaisenaan periaatteita 2–4, mutta nyt meillä on tilaisuus parantaa sitä ja miettiä, mihin periaatteet perustuvat.

Parannus 1
Tietokanta rikkoo periaatetta 2, koska sarakkeessa kaverit on lista kavereista. Tällaisen sarakkeen ongelmana on, että meidän on vaivalloista käsitellä sitä SQL-komennoilla. Esimerkiksi miten voisimme selvittää, keiden käyttäjien listoilla Maija esiintyy?

Ratkaisu on luopua sarakkeesta kaverit ja luoda sen sijaan uusi taulu Kaverit, jonka jokainen rivi ilmaisee yhden kaverisuhteen muotoa "käyttäjän x listalla on kaveri y":

kayttaja    kaveri
----------  ----------
uolevi      maija
uolevi      liisa
maija       aapeli
aapeli      uolevi
aapeli      maija
Tämän jälkeen taulu Kayttajat näyttää seuraavalta:

id          tunnus      yhteensa
----------  ----------  ----------
1           uolevi      2
2           maija       1
3           liisa       0
4           aapeli      2
Nyt voimme selvittää näin helposti, keiden listoilla Maija esiintyy:

SELECT kayttaja FROM Kaverit WHERE kaveri='maija';
Parannus 2
Uusi taulu Kaverit on kätevä, mutta se rikkoo periaatetta 3, koska käyttäjien tunnukset esiintyvät nyt monessa paikassa tietokannassa. Tässä on ongelmana, että jos tunnus muuttuu, meidän täytyy metsästää kaikki paikat, joissa tunnus esiintyy.

Ratkaisu on muuttaa taulua Kaverit niin, että siinä käytetään viittauksia. Taulun uudeksi sisällöksi tulee:

kayttaja_id  kaveri_id
-----------  ----------
1            2
1            3
2            4
4            1
4            2
Huomaa, että tämän seurauksena on selvästi vaikeampaa etsiä, keiden listoilla Maija esiintyy, koska meidän pitää hakea tunnukset taulusta Kayttajat:

SELECT A.tunnus
FROM Kayttajat A, Kayttajat B, Kaverit K
WHERE A.id = K.kayttaja_id AND B.id = K.kaveri_id AND B.tunnus = 'maija';
Tästä huolimatta muutos on kokonaisuutena kannattava, koska nyt jokainen tunnus esiintyy vain yhdessä paikassa taulussa Kayttajat.

Parannus 3
Tietokanta rikkoo vielä periaatetta 4, koska taulun Kayttajat sarakkeen yhteensa arvo on laskettavissa taulun Kaverit avulla. Sinänsä tämä on kätevä sarake, koska voimme laskea vaikkapa Uolevin kaverit näin:

SELECT yhteensa FROM Kayttajat WHERE tunnus='uolevi';
Ongelmana on kuitenkin, että aina kun muutamme kavereita, meidän pitäisi myös päivittää tilanne sarakkeeseen yhteensa. Parempi ratkaisu on poistaa sarake taulusta:

id          tunnus    
----------  ----------
1           uolevi    
2           maija     
3           liisa     
4           aapeli    
Vaikka saraketta ei ole enää, voimme laskea kavereiden määrän näin:

SELECT COUNT(*)
FROM Kayttajat A, Kaverit K
WHERE A.tunnus='uolevi' AND k.kayttaja = A.id;





Rakenteen kuvaaminen
Käsittelemme seuraavaksi kaksi tapaa tietokannan rakenteen kuvaamiseen. Graafinen tietokantakaavio näyttää tietokannan taulut, sarakkeet ja niiden väliset viittaukset, kun taas SQL-skeema kuvaa SQL-komennot, jotka luovat tietokannan taulut.

Tietokantakaavio
Tietokantakaavio on tietokannan graafinen esitys, jossa jokainen tietokannan taulu on laatikko, joka sisältää taulun nimen ja sarakkeet listana. Rivien viittaukset toisiinsa esitetään laatikoiden välisinä yhteyksinä.

Tietokantakaavion piirtämiseen on monia vähän erilaisia tapoja. Seuraava kaavio on luotu netissä olevalla työkalulla dbdiagram.io:


Tässä merkki 1 tarkoittaa, että sarakkeessa on eri arvo joka rivillä, ja merkki * puolestaan tarkoittaa, että sarakkeessa voi olla sama arvo usealla rivillä. Esimerkiksi taulussa Tuotteet joka rivillä on eri id, mutta taulussa Ostokset monella rivillä voi olla sama tuote_id.

SQL-skeema
SQL-skeema on tietokannan rakenteen tekstimuotoinen esitys, jossa annetaan tietokannan luomiseen tarvittavat SQL-komennot. Tämän esitystavan etuna on, että se on varmasti täsmällinen ja voimme halutessamme luoda suoraan tietokannan sen perusteella.

Esimerkiksi tässä on äskeistä tietokantaa vastaava SQL-skeema:

CREATE TABLE Tuotteet (id INTEGER PRIMARY KEY, nimi TEXT, hinta INTEGER);
CREATE TABLE Asiakkaat (id INTEGER PRIMARY KEY, nimi TEXT);
CREATE TABLE Ostokset (tuote_id INTEGER, asiakas_id INTEGER);
Jos oletamme, että tämä skeema on tiedostossa kuvaus.sql, voimme luoda tietokannan SQLite-tulkissa seuraavasti komennolla .read:

sqlite> .read kuvaus.sql
sqlite> .tables
Asiakkaat  Ostokset   Tuotteet
Toisaalta voimme myös käyttää SQLite-tulkissa komentoa .schema, joka antaa nykyisen tietokannan skeeman:

sqlite> .schema
CREATE TABLE Tuotteet (id INTEGER PRIMARY KEY, nimi TEXT, hinta INTEGER);
CREATE TABLE Asiakkaat (id INTEGER PRIMARY KEY, nimi TEXT);
CREATE TABLE Ostokset (tuote_id INTEGER, asiakas_id INTEGER);





Muuttuva tietokanta
Käytännössä käy harvoin niin, että tietokanta suunnitellaan ensin valmiiksi ja sitten sen rakenne säilyy samanlaisena tuomiopäivään asti. Paljon tavallisempaa on, että tietokannan rakenne muuttuu silloin tällöin.

Muutosten toteutus
Yksinkertainen muutos on, että tietokantaan ilmestyy uusi taulu. Tässä tilanteessa voimme luoda taulun komennolla CREATE TABLE, kuten yleensäkin.

Voimme myös muuttaa olemassa olevan taulun rakennetta komennolla ALTER TABLE. Tällä komennolla on monta eri käyttötapaa riippuen siitä, mitä haluamme tehdä. Esimerkiksi voimme lisätä tauluun uuden sarakkeen liittämällä komentoon ADD COLUMN.

Tarkastellaan esimerkkinä seuraavaa taulua Asiakkaat:

id           nimi
-----------  ----------
1            Uolevi
2            Maija
3            Aapeli
Kun haluamme lisätä tauluun uuden sarakkeen osoite, voimme suorittaa seuraavan komennon:

ALTER TABLE Asiakkaat ADD COLUMN osoite TEXT;
Tämän seurauksena taulun sisällöksi tulee:

id           nimi        osoite
-----------  ----------  ----------
1            Uolevi
2            Maija
3            Aapeli
Koska tauluun lisättiin uusi sarake, olemassa olevilla riveillä ei ole mitään tietoa siinä. Tietoa voi kuitenkin päivittää tämän jälkeen UPDATE-komennolla.

Komennon ALTER TABLE käyttötavat vaihtelevat tietokantajärjestelmän mukaan, ja lisätietoja löytyy käytetyn tietokannan dokumentaatiosta. SQLitessä komento on varsin rajoittunut verrattuna esimerkiksi PostgreSQL:ään.

Muuttamisen haasteet
Olemassa olevan tietokannan rakenteen muuttamisessa on yksi ongelma: tietokannassa on yleensä jo valmiina tietoa ja sitä käytetään jatkuvasti jossain järjestelmässä. Kuinka toteuttaa muutokset niin, että ne eivät häiritse järjestelmän toimintaa?

Taulun ja sarakkeen lisääminen ovat usein helppoja muutoksia, koska ne eivät haittaa tietokannan käyttämistä vanhalla tavalla, mutta vaikeampia muutoksia ovat esimerkiksi sarakkeen poistaminen tai sen nimen muuttaminen.

Yksi hyvä periaate on tehdä muutoksia vaiheittain. Esimerkiksi jos sarakkeen nimeä täytyy muuttaa, voimme toimia seuraavaan tapaan:

Lisätään tauluun uusi sarake vanhan sarakkeen rinnalle.
Muutetaan tietoa tallentavia SQL-komentoja niin, että ne tallentavat tiedon sekä vanhaan että uuteen sarakkeeseen.
Kopioidaan nykyisillä riveillä tiedot vanhasta sarakkeesta uuteen sarakkeeseen.
Muutetaan tietoa lukevia SQL-komentoja niin, että ne lukevat tiedon uudesta sarakkeesta.
Muutetaan tietoa tallentavia SQL-komentoja niin, että ne tallentavat tiedon vain uuteen sarakkeeseen.
Poistetaan vanha sarake taulusta.
Tämän ansiosta järjestelmä voi käyttää tietokantaa koko ajan, eikä järjestelmän käyttäjä huomaa muutosta. Prosessin päätteeksi sarakkeen nimi on muuttunut toiseksi.

Migraatio
Tietokantojen yhteydessä migraatio voi tarkoittaa tietokannan rakenteen muuttumista tai tietokannan sisällön siirtämistä uuteen paikkaan. Tutustumme asiaan käytännössä tarkemmin myöhemmillä kursseilla, jossa luodaan tietokantaa käyttäviä sovelluksia
