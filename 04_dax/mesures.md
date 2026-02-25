Ce fichier contient les mesures DAX utilisées dans ce  rapport, ainsi que leur définition textuelle précise.

## Calculs du chiffre d'affaire
- Calcul du chiffre d'affaire dans le contexte de filtre
```
CA = 
CALCULATE(
    SUM(planity_rdv_36_mois[montant_recu_eur])
)
```

- Calcul du chiffre d'affaire du derniers mois civil terminé
```
CA M-1 = 
VAR EndDate   = [Date Fin M-1 (today)]
VAR StartDate = DATE(YEAR(EndDate), MONTH(EndDate), 1)
RETURN
CALCULATE(
    [CA],
    DATESBETWEEN(Calendrier[Date], StartDate, EndDate)
)
```


```
CA M-1 N-1 = 
VAR EndDate   = EOMONTH(TODAY(), -13)
VAR StartDate = DATE(YEAR(EndDate), MONTH(EndDate), 1)
RETURN
CALCULATE(
    [CA],
    DATESBETWEEN(Calendrier[Date], StartDate, EndDate)
)
```
