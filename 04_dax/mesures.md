Ce fichier contient les mesures DAX utilisées dans ce rapport, ainsi que leur définition précise. Les mesures sont triées par ordre alphabétique.

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

- Chiffre d'affaire du dernier mois civil terminé (ignore contexte de filtre)
```
CA M-1 (ignore filtres) = 
VAR EndDate   = [Date Fin M-1 (today)]
VAR StartDate = DATE(YEAR(EndDate), MONTH(EndDate), 1)
RETURN
CALCULATE(
    [CA],
    REMOVEFILTERS(Calendrier),
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

- Chiffre d'affaire du dernier mois civil terminé 1 an plus tôt (ignore le contexte de filtre)
```
CA M-1 N-1 (ignore filtres) = 
VAR EndDate   = EOMONTH(TODAY(), -13)
VAR StartDate = DATE(YEAR(EndDate), MONTH(EndDate), 1)
RETURN
CALCULATE(
    [CA],
    REMOVEFILTERS(Calendrier),
    DATESBETWEEN(Calendrier[Date], StartDate, EndDate)
)
- Chiffre d'affaire de l'avant-dernier mois civil terminé dans le contexte de filtre
```

- Chiffre d'affaire de l'avant-dernier mois civil
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

- Chiffre d'affaire de l'avant-dernier mois civil (ignore le contexte de filtre)
```
CA M-2 (ignore filtres) = 
VAR EndDate   = EOMONTH(TODAY(), -2)
VAR StartDate = DATE(YEAR(EndDate), MONTH(EndDate), 1)
RETURN
CALCULATE(
    [CA],
    REMOVEFILTERS(Calendrier),
    DATESBETWEEN(Calendrier[Date], StartDate, EndDate)
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

- Nombre de clients actifs projetés sur les 6 prochains mois (sont pris en compte : les rdv planifiés, la taux d'abandon et les nouveaux clients, et la saisonnalité)
```
Clients actifs projetés = 
VAR Pivot = [Date Pivot]
VAR d = MAX(Calendrier[Date])
VAR N = MAX(0, DATEDIFF(EOMONTH(Pivot,0), EOMONTH(d,0), MONTH))

VAR ActifsNow =
    CALCULATE(
        DISTINCTCOUNT(planity_rdv_36_mois[client_id]),
        planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"},
        DATESINPERIOD(Calendrier[Date], Pivot, -1, MONTH)
    )
VAR NewAvg = [Nouveaux clients - moy / mois (12M)]
VAR Churn = [Taux abandon - moy mensuel (12M)]
VAR Ret = 1 - Churn
RETURN
IF(
    Churn = 0,
    ActifsNow + NewAvg * N,
    ActifsNow * POWER(Ret, N) + NewAvg * (1 - POWER(Ret, N)) / Churn
)
```

- Date du premier jour du dernier moi civil terminé
```
Date Début M-1 (today) = 
VAR EndDate = [Date Fin M-1 (today)]
RETURN DATE(YEAR(EndDate), MONTH(EndDate), 1)
```

- Dernière date du dernier mois civil terminé
```
Date Dernier Mois Terminé = EOMONTH(TODAY(), -1)
```

- Date du dernier jour du dernier moi civil terminé
```
Date Fin M-1 (today) = 
EOMONTH(TODAY(), -1)
```

- Date actuelle (pour date pivot)
```
Date Pivot = 
MAXX(ALLSELECTED(Calendrier[Date]), Calendrier[Date])
```

- Durée moyenne de suivi d’un client, en mois, entre son premier et son dernier rendez-vous réalisé, en ignorant les filtres de date.
```
Durée traitement moyenne (mois) (ignore filtres) = 
VAR Clients =
    VALUES(clients_36_mois[client_id])
RETURN
AVERAGEX(
    Clients,
    VAR Client = clients_36_mois[client_id]
    VAR FirstDate =
        CALCULATE(
            MIN(planity_rdv_36_mois[date]),
            REMOVEFILTERS(Calendrier),
            planity_rdv_36_mois[client_id] = Client,
            planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"}
        )
    VAR LastDate =
        CALCULATE(
            MAX(planity_rdv_36_mois[date]),
            REMOVEFILTERS(Calendrier),
            planity_rdv_36_mois[client_id] = Client,
            planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"}
        )
    RETURN
        IF(
            ISBLANK(FirstDate) || ISBLANK(LastDate),
            BLANK(),
            DIVIDE(DATEDIFF(FirstDate, LastDate, DAY), 30.4375)
        )
)
```

- Facteur de saisonnalité calculé sur 1 an, sensibilité à la semaine
```
Facteur saisonnalité (semaine) - lissé = 
VAR Pivot = [Date Pivot]
VAR WeekNum = MAX(Calendrier[Numero de la semaine])
VAR Window52W =
    DATESINPERIOD(Calendrier[Date], Pivot, -364, DAY)
VAR Heures3Weeks =
    CALCULATE(
        [Heures RDV réalisés],
        FILTER(
            Window52W,
            VAR w = WEEKNUM(Calendrier[Date], 21)
            RETURN w IN { WeekNum - 1, WeekNum, WeekNum + 1 }
        )
    )
VAR AvgWeekly =
    DIVIDE(
        CALCULATE([Heures RDV réalisés], Window52W),
        52
    )
RETURN
DIVIDE(Heures3Weeks / 3, AvgWeekly)
```

- Heures attentdues non calées sur la base des 12 derniers mois, en prenant en compte les rdv prévus et la saisonnalité.
```
Heures attendues (non calées) = 
[Clients actifs projetés] * [Heures par client actif (12M)] * [Facteur saisonnalité (semaine) - lissé]
```

- Temps non facturé par séance (administratif, échange client)
```
Heures non facturées (admin) = 
DIVIDE(
    CALCULATE(
        SUM(planity_rdv_36_mois[temps_non_facture_min])
    ),
    60
)
```


- Total d'heures d'ouverture théoriques du cabinet dans le contexte de filtre (hors congés, jours fériés, fermetures exceptionnelles)
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

- Heures de soins réalisées par client actifs au cours des 12 derniers mois
```
Heures par client actif (12M) = 
VAR Pivot = [Date Pivot]
VAR Heures = [Heures RDV réalisés - 12M]
VAR ClientsActifs =
    CALCULATE(
        DISTINCTCOUNT(planity_rdv_36_mois[client_id]),
        planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"},
        DATESINPERIOD(Calendrier[Date], Pivot, -12, MONTH)
    )
RETURN
DIVIDE(Heures, ClientsActifs)
```

- Heures réellement réalisées dans le passé et, pour le futur, rendez-vous déjà planifiés avec une estimation des heures supplémentaires attendues
```
Heures RDV (réel + calés + prévision) = 
VAR Pivot = [Date Pivot]
VAR d = MAX(Calendrier[Date])
RETURN
IF(
    d <= Pivot,
    [Heures RDV réalisés], -- Nombre d'heures de prestation réalisées (hors temps administratif et temps sans client)
    [Heures RDV calées] + [Heures attendues (non calées)]
)
```

- Heures de RDV annulés, ou le/la client.e ne s'est pas présenté.e
```
Heures RDV annulés = 
DIVIDE(
    CALCULATE(
        SUM(planity_rdv_36_mois[duree_prestation_min]),
        planity_rdv_36_mois[statut] = "Annulé"
    ),
    60
)
```

- Nombre d'heures de soins effecrifs calées à partir de la date pivot
```
Heures RDV calées = 
VAR Pivot = [Date Pivot]
RETURN
CALCULATE(
    SUM(planity_rdv_36_mois[durée_prestation_heure]),            -- ou ta mesure d'heures
    planity_rdv_36_mois[date] > Pivot,
    planity_rdv_36_mois[statut] IN {"Confirmé","Planifié","À venir"}
)
```

- Nombre d'heures de soins effectifs pour lesquelles le/la client.es ne s'est pas présenté.e
```
Heures RDV no-show = 
DIVIDE(
    CALCULATE(
        SUM(planity_rdv_36_mois[duree_prestation_min]),
        planity_rdv_36_mois[statut] = "No-show"
    ),
    60
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

- Valeur de vie client calculée sur les 12 premiers mois après la première consultation. Seuls les clients ayant réalisés leur première consultation 12 mois auparavant minimum sont pris en compte. **Q : combien de temps max dure un traitement ? certains clients font-ils plusieurs zones ?**
```
LTV estimée (12 mois après 1re consult) = 
VAR ClientsCohorte =
    VALUES(clients_36_mois[client_id])
RETURN
AVERAGEX(
    ClientsCohorte,
    VAR Client = clients_36_mois[client_id]
    VAR DateStart =
        CALCULATE(
            MIN(clients_36_mois[date_premiere_consultation]),
            clients_36_mois[client_id] = Client
        )
    VAR DateEnd = EDATE(DateStart, 12)
    RETURN
        CALCULATE(
            SUM(planity_rdv_36_mois[montant_recu_eur]),
            planity_rdv_36_mois[client_id] = Client,
            planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"},
            REMOVEFILTERS(Calendrier),
            DATESBETWEEN(Calendrier[Date], DateStart, DateEnd)
        )
)
```

- Tendance du nombre de nouveaux clients sur les 12 derniers mois
```
Nouveaux clients - moy / mois (12M) = 
VAR Pivot = [Date Pivot]
RETURN
AVERAGEX(
    DATESINPERIOD(Calendrier[Date], Pivot, -12, MONTH),
    [Nouveaux clients du mois]
)
```

- Nombre de nouveau clients enregistrés sur le dernier mois civil terminé
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
```

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

- Calcul de l'évolution mensuelle du taux d'abandon au cours de l'année dernière glissante
```
Taux abandon - moy mensuel (12M) = 
VAR Pivot = [Date Pivot]
RETURN
AVERAGEX(
    DATESINPERIOD(Calendrier[Date], Pivot, -12, MONTH),
    VAR FinM = EOMONTH(MAX(Calendrier[Date]), 0)
    VAR DebM = DATE(YEAR(FinM), MONTH(FinM), 1)
    VAR DebM1 = EOMONTH(FinM, 0) + 1
    VAR FinM1 = EOMONTH(FinM, 1)

    VAR ActifsM =
        CALCULATETABLE(
            VALUES(planity_rdv_36_mois[client_id]),
            planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"},
            DATESBETWEEN(Calendrier[Date], DebM, FinM)
        )
    VAR ActifsM1 =
        CALCULATETABLE(
            VALUES(planity_rdv_36_mois[client_id]),
            planity_rdv_36_mois[statut] IN {"Réalisé","Honoré"},
            DATESBETWEEN(Calendrier[Date], DebM1, FinM1)
        )
    VAR Retenus = COUNTROWS(INTERSECT(ActifsM, ActifsM1))
    VAR Total = COUNTROWS(ActifsM)
    RETURN 1 - DIVIDE(Retenus, Total)
)
```

- Taux d'annulation par rapport aux heures d'ouverture théoriques
```
Taux annulation = 
DIVIDE([Heures RDV annulés], [Heures ouverture théoriques])
```

- Taux d'heures occupées mais non facturées (temps d'échange, administratif) par rapport aux heures d'ouverture théoriques
```
Taux d'heures non facturées (admin) = 
    CALCULATE(
        DIVIDE(Mesures[Heures non facturées (admin)], [Heures ouverture théoriques]))
```

- Taux de remplissage par rapport aux heures d'ouverture théoriques
```
Taux de remplissage = 
DIVIDE([Heures RDV réalisés], [Heures ouverture théoriques])
```

- Taux de remplissage du dernier mois civil terminé par rapport aux heures d'ouverture théoriques (ignore le contexte de filtre)
```
Taux remplissage M-1 (ignore filtres) = 
VAR EndDate   = [Date Fin M-1 (today)]
VAR StartDate = DATE(YEAR(EndDate), MONTH(EndDate), 1)
RETURN
CALCULATE(
    [Taux de remplissage],
    REMOVEFILTERS(Calendrier),
    DATESBETWEEN(Calendrier[Date], StartDate, EndDate)
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









### Poubelle


- Cette mesure affiche le nombre d’heures de rendez-vous, en combinant réel et prévision. Pour les dates passées ou jusqu’à la date pivot sélectionnée, elle affiche les heures réellement réalisées. Pour les dates futures, elle affiche une estimation basée sur la moyenne des 8 dernières semaines, convertie en valeur journalière.
```
Heures RDV (réel + prévision) = 
VAR d = MAX(Calendrier[Date])
VAR DatePivot = MAXX(ALLSELECTED(Calendrier[Date]), Calendrier[Date])
RETURN
IF(
    d <= DatePivot,
    [Heures RDV réalisés],
    [Heures RDV réalisées - moy 8 sem] / 7   -- converti en "par jour" si ton axe est quotidien

)
```


```
Heures RDV réalisées - moy 8 sem = 
VAR DateFin = MAX(Calendrier[Date])
VAR DateDeb = DateFin - 56
RETURN
DIVIDE(
    CALCULATE(
        [Heures RDV réalisés],
        DATESBETWEEN(Calendrier[Date], DateDeb, DateFin)
    ),
    8
)
```



