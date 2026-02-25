Ce fichier contient les mesures DAX utilisées dans ce  rapport, ainsi que leur définition textuelle précise.

## Dates
- Date du premier jour du dernier moi civil terminé
```
Date Début M-1 (today) = 
VAR EndDate = [Date Fin M-1 (today)]
RETURN DATE(YEAR(EndDate), MONTH(EndDate), 1)
```

- Date du dernier jour du dernier moi civil terminé
```
Date Fin M-1 (today) = 
EOMONTH(TODAY(), -1)
```

## Chiffre d'affaire
- Chiffre d'affaire dans le contexte de filtre
```
CA = 
CALCULATE(
    SUM(planity_rdv_36_mois[montant_recu_eur])
)
```

- Chiffre d'affaire du dernier mois civil terminé
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

- Chiffre d'affaire du dernier mois civil terminé 1 an plus tôt
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

- Chiffre d'affaire de l'avant-dernier mois civil terminé dans le contexte de filtre
```
CA M-2 = 
VAR EndDate   = EOMONTH(TODAY(), -2)
VAR StartDate = DATE(YEAR(EndDate), MONTH(EndDate), 1)
RETURN
CALCULATE(
    [CA],
    DATESBETWEEN(Calendrier[Date], StartDate, EndDate)
)
```

- Variation en pourcentage du CA entre l'avant-dernier moi civil et le dernier mois civil terminé.
```
Var % MoM = 
DIVIDE(
    [CA M-1] - [CA M-2],
    [CA M-2]
)
```

- Variation en pourcentage du CA entre le dernier mois civil terminé civil 12 mois plus tôt et le dernier mois civil terminé.
```
Var % YoY = 
DIVIDE(
    [CA M-1] - [CA M-1 N-1],
    [CA M-1 N-1]
)
```

- Total des charges apparaissant sur le compte bancaire professionnel dans le contexte de fitre
```
Charges = 
ABS(
    CALCULATE(
        SUM(banque_transactions_36_mois[montant_eur]),
        banque_transactions_36_mois[type] = "Charge"
    )
)
```

- Total des revenus apparaissant sur le compte bancaire dans le contexte de filtre
```
Paiements = CALCULATE(
    SUM(banque_transactions_36_mois[montant_eur]),
    banque_transactions_36_mois[type] = "Recette"
)
```

- Résultat net des opérations sur le compte bancaire professionnel
```
Résultat net = [Paiements]-[Charges]

- Nombre d'heures d'ouverture thoériques pendant la semaine (hors congés, jours fériés, fermetures exceptionnelles)
```

## Clients
- Nombre de clients uniques ayant effectué au moins un RDV réalisé ou honoré au cours des 6 derniers mois glissants.
```
Clients actifs (6M) = 
VAR DateFin = MAX(Calendrier[Date])
VAR DateDebut = EDATE(DateFin, -6)
RETURN
CALCULATE(
    DISTINCTCOUNT(planity_rdv_36_mois[client_id]),
    planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"},
    DATESBETWEEN(Calendrier[Date], DateDebut, DateFin)
)
```

- Total d'heures douvertures théoriques du cabinet dans le contexte de filtre (hors congés, jours fériés, fermetures exceptionnelles)
```
Heures ouverture théoriques = 
SUMX(
    VALUES(Calendrier[Date]),
    VAR dow = WEEKDAY(Calendrier[Date], 2)  -- 1=Lun ... 7=Dim
    RETURN
        SWITCH(
            TRUE(),
            dow >= 1 && dow <= 5, 10,
            dow = 6, 5,
            0
        )
)
```

- Nombre d'heures de prestation réalisées (hors temps administratif et temps sans client)
```
Heures RDV réalisés = 
DIVIDE(
    CALCULATE(
        SUM(planity_rdv_36_mois[duree_prestation_min]),
        planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"}
    ),
    60
)
```

- Taux de remplissage du cabinet 
```
Taux de remplissage = 
DIVIDE([Heures RDV réalisés], [Heures ouverture théoriques])
```

- Nombre de nouveau clients enregistrés sur le mois
```
Nouveaux clients du mois = 
VAR MoisDebut = DATE(YEAR(MAX(Calendrier[Date])), MONTH(MAX(Calendrier[Date])), 1)
VAR MoisFin = EOMONTH(MAX(Calendrier[Date]), 0)
RETURN
CALCULATE(
    DISTINCTCOUNT(planity_rdv_36_mois[client_id]),
    FILTER(
        VALUES(planity_rdv_36_mois[client_id]),
        VAR FirstVisit =
            CALCULATE(
                MIN(planity_rdv_36_mois[date]),
                planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"},
                ALL(Calendrier)
            )
        RETURN FirstVisit >= MoisDebut && FirstVisit <= MoisFin
    )
)
```

## Titres et sous-titres dynamiques
- "+/- x% par rapport à M-1", soit l'évolution en pourcentage du chiffre d'ffaire du dernier moi civil terminé par rapport à l'avant-dernier mois civil
```
Sous-titre Var MoM = 
VAR VarMoM = [Var % MoM]
VAR Sign = IF(VarMoM > 0, "+", "")
VAR TxtPct =
    IF(
        ISBLANK(VarMoM),
        "—",
        Sign & FORMAT(VarMoM, "0%")
    )
RETURN
IF(
    ISBLANK(VarMoM),
    "Pas de comparaison disponible",
    TxtPct & " VS M-1")
```

- "+/- x% par rapport à Y-1", soit l'évolution en pourcentage du chiffre d'ffaire du dernier moi civil terminé par rapport au même mois de la dernière année civile
```
Sous-titre Var YoY = 
VAR VarYoY = [Var % YoY]
VAR Sign = IF(VarYoY > 0, "+", "")
VAR TxtPct =
    IF(
        ISBLANK(VarYoY),
        "—",
        Sign & FORMAT(VarYoY, "0%")
    )
RETURN
IF(
    ISBLANK(VarYoY),
    "Pas de comparaison Y-1 disponible",
    TxtPct & " par rapport à Y-1"
)
```

- Nom du dernier mois civil terminé
```
Titre Dernier Mois Terminé = 
FORMAT(
    [Date Dernier Mois Terminé],
    "mmmm"
)
```
