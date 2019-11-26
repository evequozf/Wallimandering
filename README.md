# Wallimandering

Outil de simulation de résultats d'élection au Grand Conseil valaisan, se basant sur les résultats communaux réels d'une année électorale donnée (par ex. 2017), et permettant de faire varier le contexte de vote :

- nombre de députés (130, 100, ...)
- nombre de circonscriptions électorales (13, 6, 3, 1, ...)
- base de calcul pour la répartition régionale des sièges (population suisse, population totale, ...)
- quorum (8%, 3%, ...)

## Comment utiliser ?

Après avoir téléchargé le code, dans le répertoire courant, créer deux répertoires initialement vides :
- `./2017-resultats-bruts`
- `./2017-scenarios`

Trois scripts iPython Notebook (Python 3) sont ensuite utiles, décrits ci-dessous.

### _get-results.ipynb_ 

- télécharge les résultats de l'élection au GC de 2017, et les nettoie, plaçant des fichiers temporaires dans le répertoire `./2017-resultats-bruts`
- nettoire les résultats, les reformate (*wrangle*) en un seul fichier
- regroupe les listes électorales en groupes parlementaires (parti)
- estime la taille de l'électorat de chaque parti dans chaque commune
- exporte le résultat dans un fichier Excel dans le répertoire courant `2017-resultats-communes.xlsx`

### _augment-results.ipynb_

- lit le fichier `2017-resultats-communes.xlsx`
- l'augmente avec des données statistiques tirées du fichier `population-vs-2018.xlsx` (chiffres issus de l'office valaisan de statistique, reformatés à la main)
    - numéro OFS de la commune
    - population suisse et étrangère et de chaque genre dans la commune
- exporte le résultat dans le fichier `2017-resultats-communes-augmente.xlsx`

### _simulations.ipynb_

Fichier principal de simulation, permettant de simuler le résultat d'une élection en se basant sur les scores obtenus à l'élection de 2017 (fichier `2017-resultats-communes-augmente.xlsx` préparés selon instructions précédentes), en faisant varier les paramètres:

- nombre de députés (130, 100, ...)
- découpage électoral (circonscriptions, construites par regroupement des communes existantes ou des 13 districts historiques)
- base de calcul pour la répartition régionale des sièges (population suisse, population totale, ...)
- quorum (8%, 3%, ...)

#### Exemples

1. Simuler une élection sur la base de la répartition en 6 arrondissements (scénario ayant effectivement eu lieu en 2017). On crée pour ce faire un `dict` qui fait correspondre à chaque district historique son arrondissement, et on le passe en argument de la fonction principale. Par ailleurs, la base de calcul pour la répartition des sièges est la population suisse de l'arrondissement (`col_proportion=Suisses`), on élit 130 députés (`num_deputes=130`) et le quorum est fixé à 8% (`quorum=0.08`) : il s'agit des circonstances effectivement appliquées pour cette élection.

        arrondissements = {
            'Goms':'ArrBrig',
            'Östlich Raron':'ArrBrig',
            'Brig':'ArrBrig', 
            'Visp':'ArrVisp',
            'Westlich Raron':'ArrVisp',
            'Leuk':'ArrVisp',
            'Sierre':'ArrSierre',
            'Hérens':'ArrSion',
            'Sion':'ArrSion',
            'Conthey':'ArrSion',
            'Martigny':'ArrMy',
            'Entremont':'ArrMy',
            'St-Maurice':'ArrMon',
            'Monthey':'ArrMon'
        }
        scenario('2017-resultats-communes-augmente.xlsx',
                 './2017-scenarios/',
                 '6Arrondissements',
                 num_deputes=130,
                 map_districts = arrondissements,
                 col_proportion='Suisses',
                 quorum=0.08)

    Cette exécution crée le fichier `./2017-scenarios/6Arrondissements-N130-C6-PSuisses-Q8.0.xlsx` avec les résultats partisans par district, et retourne le nombre de sièges de chaque parti:

        {'District': 'ArrBrigArrMonArrMyArrSierreArrSionArrVisp',
         'Scénario': '6Arrondissements-N130-C6-PSuisses-Q8.0',
         'Sièges CSP': 11.0,
         'Sièges CVP': 13.0,
         'Sièges PDC': 31.0,
         'Sièges PLR': 27.0,
         'Sièges PS': 17.0,
         'Sièges RCV': -0.0,
         'Sièges UDC': 22.0,
         'Sièges Verts': 9.0}
         
    ***ATTENTION : ce résultat est sensiblement différent de celui de l'élection "réelle" de 2017 (où le PDC a effectivement obtenu globalement 55 sièges (PDC+CVP+CSP), mais le PLR a obtenu 26 sièges (-1), l'AdG 18 (+1), l'UDC 23 (+1) et les Verts 8 (-1). Ceci est dû au fait que, afin de pouvoir généraliser la simulation à d'autres découpages électoraux, le calcul n'est pas fait sur la base des suffrages effectifs et en appliquant la méthode du double Pukelsheim, mais par estimation de la taille de l'électorat communal. Cette estimation est réalisée dans _augment-results.ipynb_, voir ce fichier pour des explications plus détaillées***
         
2. Simuler une élection sur la base de la répartition "historique" en 13 districts, soit 14 circonscriptions (car 2 demi-districts) - scénario par ailleurs "interdit" par Arrêt du Tribunal Fédéral en raison du quorum naturel trop faible dans certains districts ayant trop peu de sièges à pourvoir (districts dont le quorum naturel <10%):
            
         scenario('2017-resultats-communes-augmente.xlsx',
             './2017-scenarios/',
             'Historique',
             num_deputes=130,
             col_proportion='Suisses',
             quorum=0.08)
             
    Cette exécution crée le fichier `./2017-scenarios/Historique-N130-C14-PSuisses-Q8.0.xlsx` avec les résultats partisans par district, et retourne le nombre de sièges de chaque parti:
    
        {'District': 'BrigContheyEntremontGomsHérensLeukMartignyMontheySierreSionSt-MauriceVispWestlich RaronÖstlich Raron',
         'Scénario': 'Historique-N130-C14-PSuisses-Q8.0',
         'Sièges CSP': 9.0,
         'Sièges CVP': 14.0,
         'Sièges PDC': 31.0,
         'Sièges PLR': 26.0,
         'Sièges PS': 18.0,
         'Sièges RCV': -0.0,
         'Sièges UDC': 24.0,
         'Sièges Verts': 8.0}
         
3. Simuler une élection sur la base d'une circonscription unique, avec 100 députés et un quorum à 3%, et utiliser la population résidante (et non la population suisse) pour calculer le nombre de sièges attribué à chaque circonscription. On crée pour ce faire un `dict` qui fait correspondre la circonscription `Unique` à chaque commune et on le passe en argument de la fonction principale:

        d = pd.read_excel('2017-resultats-communes-augmente.xlsx')
        map_one_circ = {commune:'Unique' for commune,value in d['Commune'].to_dict().items() }
        scenario('2017-resultats-communes-augmente.xlsx',
                 './2017-scenarios/',
                 'Unique',
                 num_deputes=100,
                 map_communes= map_one_circ, # une seule circonscription
                 col_proportion='Population',
                 quorum=0.03)
                 
    Cette exécution crée le fichier `./2017-scenarios/Unique-N100-C1-PPopulation-Q3.0.xlsx` avec les résultats partisans par district, et retourne le nombre de sièges de chaque parti:
    
            {'District': 'Unique',
         'Scénario': 'Unique-N100-C1-PPopulation-Q3.0',
         'Sièges CSP': 8.0,
         'Sièges CVP': 10.0,
         'Sièges PDC': 24.0,
         'Sièges PLR': 20.0,
         'Sièges PS': 14.0,
         'Sièges RCV': -0.0,
         'Sièges UDC': 17.0,
         'Sièges Verts': 7.0}
