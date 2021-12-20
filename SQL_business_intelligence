# DOMANDA 1
# Distribuzione del numero degli studenti iscritti nei vari appelli,
# suddivisa per anni e per corso di laurea

## Tabella Normalizzata
SELECT count(*) as NumeroIscrizioni, YEAR(STR_TO_DATE(A.dtappello, '%d/%m/%Y')) as AnnoAppello, c.cds as CorsoDiLaurea
FROM appelli a
         INNER JOIN cds c on a.cdscod = c.cdscod
         INNER JOIN iscrizioni i on a.appcod = i.appcod
WHERE i.Iscrizione = 1
GROUP BY AnnoAppello, CorsoDiLaurea
ORDER BY NumeroIscrizioni DESC;


# converto nel formato corretto, poi prendo solo anno Y maiusc perchè anno lungo
##where ceck per controllare che lo studenti all'appello ma è superfluo. serve per un controllo futuro.




## Tabella Denormalizzata
SELECT count(*) as NumeroIscrizioni, YEAR(STR_TO_DATE(dtappello, '%d/%m/%Y')) as AnnoAppello, CdS as CorsoDiLaurea
FROM bos_denormalizzato
WHERE Iscrizione = 1
GROUP BY AnnoAppello, CorsoDiLaurea
ORDER BY NumeroIscrizioni DESC;

##qua non servono i join 
##csmbisno i tempi di calcolo
##magari guarda le row processed per guardare le righe effettivamente esaminate
#temporary tables, rows  joins e sorting servono per giustificare il tempo

# DOMANDA 2
# Individuazione della Top-10 degli esami più difficili suddivisi per corso di studi.

# Per esame più difficile si intende l’esame che presenta il tasso di superamento complessivo minore, considerando tutti gli appelli
# dell’Anno Accademico.
# Tasso di superamento è inteso come “numero di studenti che hanno superato l’appello” (Tab. Iscrizioni col. Superamento) su
# “numero di studenti che hanno partecipato all’appello” minore.

## Tabella Normalizzata
SELECT RapportoSup, Iscrizione, Superamento, AnnoAppello, CorsoDiLaurea, NomeEsame
FROM (
	SELECT InnerRapportoSup as RapportoSup, InnerIscrizione as Iscrizione, InnerSuperamento as Superamento, InnerAnnoAppello as AnnoAppello, InnerCorsoDiLaurea as CorsoDiLaurea, InnerNomeEsame as NomeEsame, @RankEsame := IF(@CorsoDiLaureaCorrente = InnerCorsoDiLaurea, @RankEsame + 1, 1) AS RankEsame, @CorsoDiLaureaCorrente := InnerCorsoDiLaurea
	FROM(
		SELECT (SUM(i.Superamento)/SUM(i.Iscrizione))*100 as InnerRapportoSup, SUM(i.Iscrizione) as InnerIscrizione, SUM(i.Superamento) as InnerSuperamento, YEAR(STR_TO_DATE(A.dtappello, '%d/%m/%Y')) as InnerAnnoAppello, c.cds as InnerCorsoDiLaurea, ad.ad as InnerNomeEsame
		FROM appelli a
			INNER JOIN cds c on a.cdscod = c.cdscod
			INNER JOIN ad on ad.adcod = a.adcod
			INNER JOIN iscrizioni i on a.appcod = i.appcod
		WHERE i.Iscrizione = 1 AND i.Assenza = 0
		GROUP BY InnerAnnoAppello, InnerCorsoDiLaurea, InnerNomeEsame
		ORDER BY InnerCorsoDiLaurea ASC, InnerRapportoSup ASC, InnerAnnoAppello ASC, InnerIscrizione DESC
	) as InnerTable
) as RankedTable
WHERE RankEsame <= 10;
##rapporto superamento nella prima query annidata
#variabile rankesame che ho creato serve per prendere poi solo i top 10


## Tabella Denormalizzata
SELECT RapportoSup, Iscrizione, Superamento, AnnoAppello, CorsoDiLaurea, NomeEsame
FROM (
	SELECT InnerRapportoSup as RapportoSup, InnerIscrizione as Iscrizione, InnerSuperamento as Superamento, InnerAnnoAppello as AnnoAppello, InnerCorsoDiLaurea as CorsoDiLaurea, InnerNomeEsame as NomeEsame, @RankEsame := IF(@CorsoDiLaureaCorrente = InnerCorsoDiLaurea, @RankEsame + 1, 1) AS RankEsame, @CorsoDiLaureaCorrente := InnerCorsoDiLaurea
	FROM(
		SELECT (SUM(Superamento)/SUM(Iscrizione))*100 as InnerRapportoSup, SUM(Iscrizione) as InnerIscrizione, SUM(Superamento) as InnerSuperamento, YEAR(STR_TO_DATE(dtappello, '%d/%m/%Y')) as InnerAnnoAppello, CdS as InnerCorsoDiLaurea, ad as InnerNomeEsame
		FROM bos_denormalizzato
		WHERE Iscrizione = 1 AND Assenza = 0
		GROUP BY InnerAnnoAppello, InnerCorsoDiLaurea, InnerNomeEsame
		ORDER BY InnerCorsoDiLaurea ASC, InnerRapportoSup ASC, InnerAnnoAppello ASC, InnerIscrizione DESC
	) InnerTable
) RankedTable
WHERE RankEsame <= 10;

##uguale cambiano solo join

# DOMANDA 3
# Individuazione dei corsi di laurea ad elevato tasso di commitment,
# ovvero appelli di esami diversi ma del medesimo corso di laurea che si
# sono svolti nello stesso giorno

## Tabella Normalizzata
SELECT count(a.appcod) as NumeroAppelli, STR_TO_DATE(a.dtappello, '%d/%m/%Y') as DataAppello, c.cds as CorsoDiLaurea
FROM appelli a
         INNER JOIN cds c on a.cdscod = c.cdscod
GROUP BY DataAppello, CorsoDiLaurea
ORDER BY NumeroAppelli DESC;

## Tabella Denormalizzata
SELECT count(DISTINCT AD, docente) as NumeroAppelli, STR_TO_DATE(dtappello, '%d/%m/%Y') as DataAppello, CdS as CorsoDiLaurea
FROM bos_denormalizzato
GROUP BY DataAppello, CorsoDiLaurea, docente
ORDER BY NumeroAppelli DESC;
#nella prima avevo un codice univoco per ogni appello, nella sec abbiamo dedotto che l'univocità dell'appello è 
#data da attività didattica e docente distinti



# DOMANDA 4
# Individuazione della Top-3 degli esami con media voti maggiore e minore rispettivamente, calcolati per ogni singolo corso di studi

## Tabella Normalizzata
SELECT CorsoDiLaurea, MediaVotiMaggiori as MediaVoti, DataAppello, AD
FROM (
	SELECT MediaMaggiori as MediaVotiMaggiori, CorsoDiLaureaMaggiori as CorsoDiLaurea, @RankVotiMaggiori := IF(@CorsoDiLaureaCorrente = CorsoDiLaureaMaggiori, @RankVotiMaggiori + 1, 1) AS RankVotiMaggiori, @CorsoDiLaureaCorrente := CorsoDiLaureaMaggiori, DataAppelloMaggiori as DataAppello, ADMaggiori as Ad
	FROM(
		SELECT AVG(i.Voto) as MediaMaggiori, a.appcod as CodiceAppelloMaggiori, c.cds as CorsoDiLaureaMaggiori, STR_TO_DATE(dtappello, '%d/%m/%Y') as DataAppelloMaggiori, ad.ad as ADMaggiori
		FROM appelli a
				 INNER JOIN cds c on a.cdscod = c.cdscod
				 INNER JOIN iscrizioni i on a.appcod = i.appcod
				 INNER JOIN ad ad on a.adcod = ad.adcod
		WHERE i.Iscrizione = 1 AND i.Assenza = 0 AND i.Ritiro = 0
		GROUP BY CodiceAppelloMaggiori
		ORDER BY CorsoDiLaureaMaggiori ASC, MediaMaggiori DESC
	) VotiMaggiori
) RankedTableMaggiori
WHERE RankVotiMaggiori <= 3
UNION ALL
SELECT CorsoDiLaurea, MediaVotiMinori as MediaVoti, DataAppello, AD
FROM (
	SELECT MediaMinori as MediaVotiMinori, CorsoDiLaureaMinori as CorsoDiLaurea, @RankVotiMinori := IF(@CorsoDiLaureaCorrenteMinori = CorsoDiLaureaMinori, @RankVotiMinori + 1, 1) AS RankVotiMinori, @CorsoDiLaureaCorrenteMinori := CorsoDiLaureaMinori, DataAppelloMinori as DataAppello, ADMinori as Ad
	FROM(
		SELECT AVG(i.Voto) as MediaMinori, a.appcod as CodiceAppelloMinori, c.cds as CorsoDiLaureaMinori, STR_TO_DATE(dtappello, '%d/%m/%Y') as DataAppelloMinori, ad.ad as ADMinori
		FROM appelli a
				 INNER JOIN cds c on a.cdscod = c.cdscod
				 INNER JOIN iscrizioni i on a.appcod = i.appcod
				 INNER JOIN ad ad on a.adcod = ad.adcod
		WHERE i.Iscrizione = 1 AND i.Assenza = 0 AND i.Ritiro = 0 AND i.Voto IS NOT NULL
		GROUP BY CodiceAppelloMinori
		ORDER BY CorsoDiLaureaMinori ASC, MediaMinori ASC
	) VotiMinori
) RankedTableMaggiori
WHERE RankVotiMinori <= 3
ORDER BY CorsoDiLaurea ASC, MediaVoti DESC;

##ho fatto l'unione perchè chiedeva insieme voti maggiori e minori
##attenzione che se c'è solo un voto il voto minore e quello maggiore coincidono
##quando ce ne sono minori di sei in generale ci sono i valori ripetuti
##nei voti minori confronto anche che non sia null

## Tabella Denormalizzata
SELECT CorsoDiLaurea, MediaVotiMaggiori as MediaVoti, DataAppello, AD
FROM (
	SELECT MediaMaggiori as MediaVotiMaggiori, CorsoDiLaureaMaggiori as CorsoDiLaurea, @RankVotiMaggiori := IF(@CorsoDiLaureaCorrente = CorsoDiLaureaMaggiori, @RankVotiMaggiori + 1, 1) AS RankVotiMaggiori, @CorsoDiLaureaCorrente := CorsoDiLaureaMaggiori, DataAppelloMaggiori as DataAppello, ADMaggiori as Ad
	FROM(
		SELECT AVG(Voto) as MediaMaggiori, cds as CorsoDiLaureaMaggiori, STR_TO_DATE(dtappello, '%d/%m/%Y') as DataAppelloMaggiori, ad as ADMaggiori, docente as DocenteMaggiori
		FROM bos_denormalizzato
		WHERE Iscrizione = 1 AND Assenza = 0 AND Ritiro = 0
		GROUP BY ADMaggiori, DocenteMaggiori, CorsoDiLaureaMaggiori, DataAppelloMaggiori
		ORDER BY CorsoDiLaureaMaggiori ASC, MediaMaggiori DESC
	) VotiMaggiori
) RankedTableMaggiori
WHERE RankVotiMaggiori <= 3
UNION ALL
SELECT CorsoDiLaurea, MediaVotiMinori as MediaVoti, DataAppello, AD
FROM (
	SELECT MediaMinori as MediaVotiMinori, CorsoDiLaureaMinori as CorsoDiLaurea, @RankVotiMinori := IF(@CorsoDiLaureaCorrenteMinori = CorsoDiLaureaMinori, @RankVotiMinori + 1, 1) AS RankVotiMinori, @CorsoDiLaureaCorrenteMinori := CorsoDiLaureaMinori, DataAppelloMinori as DataAppello, ADMinori as Ad
	FROM(
		SELECT AVG(Voto) as MediaMinori, cds as CorsoDiLaureaMinori, STR_TO_DATE(dtappello, '%d/%m/%Y') as DataAppelloMinori, ad as ADMinori, docente as DocenteMinori
		FROM bos_denormalizzato
		WHERE Iscrizione = 1 AND Assenza = 0 AND Ritiro = 0 AND Voto IS NOT NULL
		GROUP BY ADMinori, DocenteMinori, CorsoDiLaureaMinori, DataAppelloMinori
		ORDER BY CorsoDiLaureaMinori ASC, MediaMinori ASC
	) VotiMinori
) RankedTableMaggiori
WHERE RankVotiMinori <= 3
ORDER BY CorsoDiLaurea ASC, MediaVoti DESC;

##anche qui non c'è codice appello, quindi devo fare il raggruppamento in base al docente



# DOMANDA 5
# Calcolare la distribuzione degli studenti “fast&furious” per corso di
# studi, ovvero studenti con il rapporto “votazione media riportata negli
# esami superati” su “periodo di attività” maggiore.
# Per periodo di attività si intende il numero di giorni trascorsi tra il primo
# appello sostenuto (non necessariamente superato) e l’ultimo

## Tabella Normalizzata
SELECT i.studente as Studente, c.cds as CorsoDiLaurea, (AVG(i.Voto) / (IF(MAX(STR_TO_DATE(a.dtappello, '%d/%m/%Y')) - MIN(STR_TO_DATE(a.dtappello, '%d/%m/%Y')) = 0, 1, MAX(STR_TO_DATE(a.dtappello, '%d/%m/%Y')) - MIN(STR_TO_DATE(a.dtappello, '%d/%m/%Y'))))) as RapportoVotoPeriodo
FROM appelli a
	 INNER JOIN cds c on a.cdscod = c.cdscod
	 INNER JOIN iscrizioni i on a.appcod = i.appcod
WHERE i.Iscrizione = 1 AND i.Assenza = 0 AND i.Ritiro = 0 AND i.Voto IS NOT NULL
GROUP BY Studente, CorsoDiLaurea
ORDER BY RapportoVotoPeriodo ASC, CorsoDiLaurea ASC;
	
## Tabella Denormalizzata
SELECT Studente, CdS, (AVG(Voto) / (IF(MAX(STR_TO_DATE(dtappello, '%d/%m/%Y')) - MIN(STR_TO_DATE(dtappello, '%d/%m/%Y')) = 0, 1, MAX(STR_TO_DATE(dtappello, '%d/%m/%Y')) - MIN(STR_TO_DATE(dtappello, '%d/%m/%Y'))))) as RapportoVotoPeriodo
FROM bos_denormalizzato
WHERE Iscrizione = 1 AND Assenza = 0 AND Ritiro = 0 AND Voto IS NOT NULL
GROUP BY Studente, CdS
ORDER BY RapportoVotoPeriodo ASC, CdS ASC;

##nell'if una condizione, così se ho fatto un esame divido per 1 e non si spacca il db 
#se no verrebbe zero e non va bene
##se no prendo il valore normale


# DOMANDA 6
# Individuazione della Top-3 degli esami “trial&error”, ovvero esami che
# richiedono il maggior numero di tentativi prima del superamento.
# Dato uno corso di studi, il rispettivo valore trial&error è dato dalla
# media del numero di tentativi (bocciature) di ogni studente per ogni
# appello del corso.

## Tabella Normalizzata
SELECT NumeroInsufficienze, CorsoDiLaurea, NomeEsame
FROM (
	SELECT InnerNumeroInsufficienze as NumeroInsufficienze, InnerCorsoDiLaurea as CorsoDiLaurea, InnerNomeEsame as NomeEsame, @RankEsame := IF(@CorsoDiLaureaCorrente = InnerCorsoDiLaurea, @RankEsame + 1, 1) AS RankEsame, @CorsoDiLaureaCorrente := InnerCorsoDiLaurea
	FROM(
		SELECT (SUM(if(i.Insufficienza = 1, 1, 0)) / COUNT(*)) as InnerNumeroInsufficienze, c.cds as InnerCorsoDiLaurea, ad.ad as InnerNomeEsame
		FROM appelli a
			INNER JOIN cds c on a.cdscod = c.cdscod
			INNER JOIN ad on ad.adcod = a.adcod
			INNER JOIN iscrizioni i on a.appcod = i.appcod
		WHERE i.Iscrizione = 1 AND i.Assenza = 0
		GROUP BY InnerCorsoDiLaurea, InnerNomeEsame
		ORDER BY InnerCorsoDiLaurea ASC, InnerNumeroInsufficienze DESC
	) InnerTable
) RankedTable
WHERE RankEsame <= 3;
##visto che NON devo fare la somma di tutti i campi popolati da insufficienza
##di fatto faccio la ranked table per poter tirare fuori i top 3 di ciascun corso di laurea 

## Tabella Denormalizzata
SELECT NumeroInsufficienze, CorsoDiLaurea, NomeEsame
FROM (
	SELECT InnerNumeroInsufficienze as NumeroInsufficienze, InnerCorsoDiLaurea as CorsoDiLaurea, InnerNomeEsame as NomeEsame, @RankEsame := IF(@CorsoDiLaureaCorrente = InnerCorsoDiLaurea, @RankEsame + 1, 1) AS RankEsame, @CorsoDiLaureaCorrente := InnerCorsoDiLaurea
	FROM(
		SELECT (SUM(if(Insufficienza = 1, 1, 0)) / COUNT(*)) as InnerNumeroInsufficienze, cds as InnerCorsoDiLaurea, ad as InnerNomeEsame
		FROM bos_denormalizzato
		WHERE Iscrizione = 1 AND Assenza = 0
		GROUP BY InnerCorsoDiLaurea, InnerNomeEsame
		ORDER BY InnerCorsoDiLaurea ASC, InnerNumeroInsufficienze DESC
	) InnerTable
) RankedTable
WHERE RankEsame <= 3;



#QUERY A SCELTA
#Tabella Normalizzata

SELECT COUNT(*) as TotaleStudenti, AVG(i.Voto) as MediaVoti, s.resarea as AreaResidenza
FROM studenti s
	INNER JOIN iscrizioni i on s.studente=i.studente
GROUP BY AreaResidenza
ORDER BY MediaVoti DESC;

#Tabella Denormalizzata
SELECT COUNT(*) as TotaleStudenti, AVG(Voto) as MediaVoti, StuResArea as AreaResidenza
FROM bos_denormalizzato
GROUP BY AreaResidenza
ORDER BY MediaVoti DESC;


