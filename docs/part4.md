---
title: "More techniques"
permalink: /part4/
nav_order: 5
published: true
---

# Properties of SQL

In SQL there are many same elements as in programming: Types, statements and fucntions. We have seen many examples of SQL commands, but let's loon into the language a bit deeper.

# Types

The types and their properties depend on the database system we are using. Usually there are several options even for integers and strings. In practice though, types `INTEGER` for integers and `TEXT` for strings go a long way.

## TEXT vs. VARCHAR
Traditional SQL type for saving a string is `VARCHAR`, where we give the maximum length of the string. For example `VARCHAR(100)` means a string, which can have at most 100 characters.

This is one remainder of the programming from old times: back then the strings were often saved as an array, with a fixed amount of characters. In practice `TEXT` is more versatile as we do not have to come up with the maximum amount of characters.

## DATE, DATETIME, TIME, TIMESTAMP...

Very useful types of data are the types used for saving `DATE` and `TIME`. The naming and usage vary from database system to another, so it is best to check the details from the documentation of the databasey system we are using.

# Statements

Statements are a part of SQL command, with a certain value. For example in query

```sql
SELECT price FROM Products WHERE name='radish';
```

Has two statements: `price` and `name='radish'`. In this the statement `price` defines the content of the result set, and the statement `name='radish'` is a conditional statement to limit the query.

We can also use calculations and other operators the same way as in programming. For example the query

```sql
SELECT price*2 FROM Products WHERE name='radish';
```

returns the price of the radish doubled.

A good way to test how SQL statements work is to discuss with the database by doing queries which do not search information from any tables, but only calculate values for statements. This could be something like

```
sqlite> SELECT 2*(1+3);
8
sqlite> SELECT 'tes' || 'ts';
tests
sqlite> SELECT 3 < 5;
1
```

The first query calculates the value for statement `2*(1+3)`. The second query combines with the operator `||` the strings 'tes' and 'ts'. into 'tests'. Third query defines the truth value for statement `3 < 5`. The practice in SQL is that the truth value is given as an integer: `1 is true`, and `0 is false`.

Many items for SQL statements are already familar from programming:

* calculations: `+`, `-`, `*`, `/`, `%`
* comparison: `=`, `<>`, `<`, `<=`, `>`, `>=`
* combining conditions: `AND`, `OR`, `NOT`

In addition to these SQL has some more special features, whose knowledge is useful. Next we will look into some of them:

## BETWEEN
Statement `X BETWEEN a AND b` is true, if `X` is at least `a` and at most `b`. For example the query

```sql
SELECT * FROM Products WHERE price BETWEEN 4 AND 6;
```

Returns the products, whose price is at least 6 and at maximum 6. We can also write the same query like this:

```sql
SELECT * FROM Products WHERE price >= 4 AND price <= 6;
```

## CASE
A `CASE` structure enables conditional statements. It can have one or more `WHEN` and a possible `ELSE`. For example the query

```sql
SELECT name, 
  CASE WHEN price>5 THEN 'expensive' 
    ELSE 'cheap' 
  END 
FROM Products;
```

Retrieves the name for all the products and the information about them, if they are expensive or cheap. In this query the product is expensive if the price is over 5, otherwise it is cheap.

## IN
Statement `x IN (...)` is true, if `x` is some of the given values. For example the query

```sql
SELECT * FROM Products WHERE name IN ('turnip','cucumber','celery');
```

Returns the products whose name is turnip, cucumber or celery.

## LIKE
Statement `s LIKE p` is true, if the string `s` matches to the description `p`. In the description we can use special characters `_` (any single character) and `%` (any amount of any characters). For example

```sql
SELECT * FROM Products WHERE name LIKE '%er%';
```

Returns the products whose name contain the string `er` (such as cucumber and celery).

## NULL
Handling `NULL` values we have a separate syntax. For example

```sql
SELECT * FROM Products WHERE price IS NULL;
```

retrieves the products to which no price has been set, and the query

```sql
SELECT * FROM Products WHERE price IS NOT NULL;
```

retrieves the products to which the price has been set.

## Functions

As a part of statements we can have functions, just like in programming. As with types, the functions available and their usage are dependant on the used database system and additional information should be checked from the database system documentation.

Here are some useful SQLite functions:

```
name	        function
--------      ----------
ABS(x)	      returns the absolute value for x
COALESCE(...)	returns the first value from the list, which is not NULL
LENGTH(s)	    returns the length of the string s
LOWER(s)	    changes the characters in string s to lower case
MAX(x,y)	    returns the greater of integers x and y
MIN(x,y)	    returns the smaller of integers x and y
RANDOM()	    returns a random number
ROUND(x,d)	  returns x rounded to d decimals
UPPER(s)	    changers the characters in string s to upper case
```

For example the query

```sql
SELECT * FROM Products WHERE LENGTH(name)=6;
```

Returns the products whose name are six characters long (such as carrot, radish, turnip and celery). The query

```sql
SELECT * FROM Products ORDER BY RANDOM();
```

Returns the all the rows in random order, since the order is not based on any column but on a random number.


# Subqueries


Alikysely on SQL-komennon osana oleva lauseke, jonka arvo syntyy jonkin kyselyn perusteella. Voimme rakentaa alikyselyjä samaan tapaan kuin varsinaisia kyselyjä ja toteuttaa niiden avulla hakuja, joita olisi vaikea saada aikaan muuten.

Esimerkki
Tarkastellaan esimerkkinä tilannetta, jossa tietokannassa on pelaajien tuloksia taulussa Tulokset. Oletamme, että taulun sisältö on seuraava:

id          name        tulos     
----------  ----------  ----------
1           Uolevi      120       
2           Maija       80        
3           Liisa       120       
4           Aapeli      45        
5           Kaaleppi    115    
Haluamme nyt selvittää ne pelaajat, jotka ovat saavuttaneet korkeimman tuloksen, eli kyselyn tulisi palauttaa Uolevi ja Liisa. Saamme tämän aikaan alikyselyllä seuraavasti:

SELECT name, tulos FROM Tulokset WHERE tulos = (SELECT MAX(tulos) FROM Tulokset);
Kyselyn tuloksena on:

name        tulos     
----------  ----------
Uolevi      120       
Liisa       120       
Tässä tapauksessa alikysely on SELECT MAX(tulos) FROM Tulokset, joka antaa suurimman taulussa olevan tuloksen eli tässä tapauksessa arvon 120. Huomaa, että alikysely tulee kirjoittaa sulkujen sisään, jotta se ei sekoitu pääkyselyyn.

Tässä on vielä vähän mutkikkaampi kysely:

SELECT name, tulos FROM Tulokset WHERE tulos >= 0.9*(SELECT MAX(tulos) FROM Tulokset);
Tämä kysely näyttää, että voimme käyttää alikyselyn antamaa arvoa lausekkeen osana samaan tapaan kuin mitä tahansa muuta arvoa. Kysely hakee pelaajat, joiden tulos on enintään 10 prosenttia huonompi kuin paras tulos:

name        tulos     
----------  ----------
Uolevi      120       
Liisa       120       
Kaaleppi    115     
Riippuva alikysely
Alikysely on mahdollista toteuttaa myös niin, että sen sisältö riippuu pääkyselyssä käsiteltävästä rivistä. Näin on seuraavassa kyselyssä:

SELECT name, tulos, (SELECT COUNT(*) FROM Tulokset WHERE tulos > T.tulos) paremmat FROM Tulokset T;
Tämän kyselyn ideana on laskea jokaiselle pelaajalle, monenko pelaajan tulos on parempi kuin pelaajan oma tulos. Esimerkiksi Maijalle vastaus on 3, koska Uolevin, Liisan ja Kaalepin tulos on parempi. Kysely antaa seuraavan tuloksen:

name        tulos       paremmat
----------  ----------  ----------
Uolevi      120         0                                                    
Maija       80          3                                                    
Liisa       120         0                                                    
Aapeli      45          4                                                    
Kaaleppi    115         2                                                   
Koska taulu Tulokset esiintyy kahdessa roolissa alikyselyssä, pääkyselyn taululle on annettu vaihtoehtoinen name T. Tämän ansiosta alikyselyssä on selvää, että halutaan laskea rivejä, joiden tulos on parempi kuin pääkyselyssä käsiteltävän rivin tulos.

Monta arvoa alikyselystä
Alikysely voi palauttaa myös useita arvoja, kunhan alikyselyn tulosta käytetään kohdassa, jossa tämä on sallittua. Näin on seuraavassa kyselyssä:

SELECT name FROM Products
WHERE id IN (SELECT tuote_id FROM Ostokset WHERE asiakas_id = 1);
Tämä kysely hakee kaikkien tuotteiden nimet asiakkaan 1 ostoskorissa. Alikysely palauttaa tuotteiden id-numerot, minkä pystyy yhdistämään IN-syntaksiin.

Huomaa, että olisimme voineet rakentaa vastaavan kyselyn myös näin:

SELECT T.name
FROM Products T, Ostokset O
WHERE T.id = O.tuote_id AND O.asiakas_id = 1;
Usein alikysely onkin vaihtoehtoinen tapa toteuttaa jokin kysely, jonka voisi tehdä yhtä hyvin myös esimerkiksi sopivasti laaditulla monen taulun kyselyllä.





Tulosrivien rajaus
SQL-kysely palauttaa oletuksena kaikki ehtoihin täsmäävät rivit, mutta voimme tarvittaessa pyytää vain osan tulostaulun riveistä. Tämä on hyödyllistä esimerkiksi sovelluksissa, joissa halutaan näyttää yhdellä sivulla vain osa haun tuloksista.

Rajaustavat
Kun lisäämme kyselyn loppuun LIMIT x, kysely antaa vain x ensimmäistä tulosriviä. Esimerkiksi LIMIT 3 tarkoittaa, että kysely antaa kolme ensimmäistä tulosriviä.

Yleisempi muoto on LIMIT x OFFSET y, mikä tarkoittaa, että haluamme x riviä kohdasta y alkaen (0-indeksoituna). Esimerkiksi LIMIT 3 OFFSET 1 tarkoittaa, että kysely antaa toisen, kolmannen ja neljännen tulosrivin.

Esimerkki
Tarkastellaan esimerkkinä kyselyä, joka hakee tuotteita halvimmasta kalleimpaan:

SELECT * FROM Products ORDER BY price;
Kyselyn tuloksena on seuraava tulostaulu:

id          name        price     
----------  ----------  ----------
3           cucumber      2
5           celery     4         
2           porkkana    5         
1           radish     7         
4           turnip      8         
Saamme haettua kolme halvinta tuotetta seuraavasti:

SELECT * FROM Products ORDER BY price LIMIT 3;
Kyselyn tulos on seuraava:

id          name        price     
----------  ----------  ----------
3           cucumber      2         
5           celery     4         
2           porkkana    5      
Seuraava kysely puolestaan hakee kolme halvinta tuotetta toiseksi halvimmasta tuotteesta alkaen:

SELECT * FROM Products ORDER BY price LIMIT 3 OFFSET 1;
Tämän kyselyn tulos on seuraava:

id          name        price     
----------  ----------  ----------
5           celery     4         
2           porkkana    5         
1           radish     7      
Rajaus alikyselyssä
Tarkastellaan vielä tilannetta, jossa haluamme laskea kolmen halvimman tuotteen yhteishinnan. Seuraava kysely ei toimi halutulla tavalla:

SELECT SUM(price) FROM Products ORDER BY price LIMIT 3;
Kysely antaa kaikkien taulun tuotteiden hintojen summan:

SUM(price)
----------
26
Ongelmana on, että kysely muodostaa tulostaulun, jossa on vain yhdellä rivillä luku 26 (kaikkien tuotteiden summa), minkä jälkeen tästä tulostaulusta haetaan kolme ensimmäistä riviä (eli käytännössä vain yksi rivi).

Ratkaisu ongelmaan on luoda alikysely, jossa haetaan kolme halvinta hintaa, ja laskea sitten summa näistä hinnoista:

SELECT SUM(price) FROM (SELECT price FROM Products ORDER BY price LIMIT 3);
Nyt kysely antaa halutun tuloksen:

SUM(price)
----------
11
