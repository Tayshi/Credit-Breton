# dictionnaire-donnees
## Power Query - Langage M
**Fusion** Table *Demande de prêt* et *Apport* (LEFT JOIN)

**Rechercher/Remplacer les erreurs de données**
- Nombres d'enfants à charges : 20 30 -> 2 3
- Situation familiale : PACS -> Pacsé

## Langage DAX
**Calcul du montant prêté (Mesure)**
`Montant_prété = SUM('Demandes de pret'[Montant opération]) - SUM('Demandes de pret'[Apport])`


**Connaître l'âge du client au jour d'aujourd'hui**
`Age_client = DATEDIFF('Situation famille'[Date de naissance], NOW(),YEAR)`


**Un verdict calculé sur critères si le prêt doit être validé ou non, 1èr prise de décision du dossier**
```dax
Score_emprunteur = 

//SI Âge du client + Durée du prêt est supérieur ou égal à 82 ans alors affecter "REFUS" à score emprunteur sinon
IF(LOOKUPVALUE('Situation famille'[Age_client], 'Situation famille'[ID_Client], 'Demandes de pret'[Numéro client]) >= 82, "REFUS", 

// SI Régularité des revenus du client est égal à 3 alors affecter "REFUS" à score emprunteur sinon
IF(LOOKUPVALUE('Situation pro'[Régularité des revenus], 'Situation pro'[Numéro client], 'Demandes de pret'[Numéro client]) == "3 : Très irréguliers", "REFUS", 

// SI Revenu mensuel moyen divisé par 3 plus le nombre d'enfants à charge est inférieur à 
// remboursement mensuel du prêt alors affecter "REFUS" à score emprunteur sinon affecter "ACCEPTE" à score emprunteur.
IF(([Montant_prété] / 'Demandes de pret'[Durée]) > 
LOOKUPVALUE('Situation pro'[Revenu mensuel moyen], 'Situation pro'[Numéro client], 'Demandes de pret'[Numéro client]) / 
(3 + LOOKUPVALUE('Situation famille'[Nombre d'enfants à charge], 'Situation famille'[ID_Client], 'Demandes de pret'[Numéro client])), "REFUS", "ACCEPTE"))
)
```
### Détails descriptifs :
*Critère 1*
>Si l’âge de l’emprunteur ajouté à la durée du prêt demandé est supérieur à 82 ans (l’espérance de vie), le prêt est systématiquement refusé, car il est fortement probable que l’emprunteur ne pourra pas rembourser le prêt.


*Critère 2*
>Si la régularité des revenus est à 3, c’est-à-dire très irrégulière, le prêt est systématiquement refusé.


*Critère 3*
>Le critère 3 comprend des étapes intermédiaires :\
Le montant précis que le Crédit Breton devra apporter à l’opération. -> `Montant_prété`\
Ensuite, la banque calcule le montant mensuel de remboursement du prêt en divisant le `Montant_prété` par la durée du prêt.\
Enfin, comparer le revenu mensuel moyen de l’emprunteur et ce remboursement mensuel, pour statuer si il ou elle aura la capacité de rembourser le prêt tous les mois. Le calcul précis demande que le montant du remboursement soit inférieur au revenu mensuel divisé par 3, plus le nombre d’enfants à charge.