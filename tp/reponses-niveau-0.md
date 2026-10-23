# Schema relationnel
## 1.
categorie (<u>id</u>, libelle)
plat (<u>id</u>, nom, prix, #id_categorie)
commande (<u>id</u>, date_commande, numero_table)
commande_ligne (<u>id</u>, #id_commande, #id_plat, quantite, prix_unitaire)

## 2.
Commande_ligne contient une ligne prix unitaire pour avoir le prix exact du plat au momoent de la commande. Si on modifie le prix du plat et que c'est cette valeur qu'on utilise le montant de la commande changera toujours.

## 3.
Si on supprime la commande, les lignes de commande seront supprimées avec tous les lignes associées à cette commande. L'instruction responsable est ON DELETE CASCADE.

## 4.
- Insertion d'une quantité négatif 
- Insertion d'une date future ou passée
- INsertion d'un numéro de table négatif

# Interrogation d'API
### a)
Code: 200 OK

    {
        "id": 7,
        "nom": "Mi sao",
        "prix": 6500,
        "id_categorie": 2,
        "categorie": "Soupe"
    },
    {
        "id": 6,
        "nom": "Soupe chinoise",
        "prix": 6000,
        "id_categorie": 2,
        "categorie": "Soupe"
    }

### b)
Code: 201 Created

    {
        "id": 12,
        "nom": "Mofo gasy",
        "prix": 300,
        "id_categorie": 1
    }

### c)
Code: 422 Unprocessable Content
    
    {
    "message": "Données invalides.",
    "errors": {
        "nom": "Le nom du plat est obligatoire.",
        "prix": "Le prix doit être un nombre entier (en Ariary).",
        "id_categorie": "Choisissez une catégorie."
                }
    }

### d)
Code: 409 Conflict

    {
        "message": "Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.",
        "errors": {}
    }

### e)
Code: 404 Not Found

    {
        "message": "Commande 999 introuvable.",
        "errors": {}
    }

## 5.
Pour la requête b, le code obtenu est 201 mais pas 200 car on procède à une création.

## 6. 
Pour la requête d, c'est PostgreSQL qui bloque la suppression à cause de la clé étrangère et l'absence de ON DELETE CASCADE.
            return $this->erreur('Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.', 409);

## 7.
curl -X POST http://localhost:8080/api/commandes \
    -H "Content-Type: application/json" \
    -d '{"numero_table": 6, "lignes": [
        {
            "id_plat": 4,
            "quantité": 2
        },
        {
            "id_plat": 10,
            "quantité": 1
        }
        ]
    }'

# Suivi de requête
| # | Étape | Fichier | Fonction / ligne |
|---|---|---|---|
| 1 | Le formulaire est soumis | `components/PlatForm.vue` | `enregistrer()` |
| 2 | La requête `POST /api/plats` est construite et envoyée | `src/api.js` | `api.plats.creer()` puis `requete()` |
| 3 | Vite transmet `/api/...` au port 8080 | `vite.config.js` | `server.proxy['/api']` |
| 4 | CodeIgniter associe `POST /api/plats` au contrôleur | `backend/app/Config/Routes.php` | `Plats::create` |
| 5 | Le contrôleur lit le corps JSON | `backend/app/Controllers/Api/Plats.php` et `ApiController.php` | `create()` puis `donneesJson()` |
| 6 | Les règles de validation sont vérifiées | `backend/app/Models/PlatModel.php` | `$validationRules`, lors de `insert()` |
| 7 | La ligne est insérée dans la base de données et le plat créé est renvoyé | `backend/app/Controllers/Api/Plats.php` | `create()` |
| 8 | Le formulaire transmet le résultat à l’écran parent | `components/PlatForm.vue` | `emit('enregistre', resultat)` |
| 9 | L’écran affiche le succès et recharge la liste des plats | `views/CarteView.vue` | `surEnregistrement()` puis `charger()` |

## 8.
Le code métier de la création d’une commande se trouve dans `backend/app/Services/CommandeService.php`, dans la méthode `creer()`
Il n’est pas dans `Commandes.php` parce que le contrôleur a pour rôle de gérer la requête HTTP et la réponse : il lit le JSON, appelle le service, transforme une `RegleMetierException` en réponse d’erreur et renvoie la commande créée. 

# Notions de vue

## 0.5 Repérer les notions de Vue

| Notion                 | À quoi ça sert                                                    | Votre exemple                                                                                                                                                                                                                            |
|------------------------|-------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ref()`                | une variable dont le changement met l’écran à jour                | frontend/src/views/CommandeView.vue:7-17<br/> const plats = ref([]) et const numeroTable = ref('') : ce sont des variables réactives, donc tout changement les met automatiquement à jour dans l’écran.                                  |
| `computed()`           | une valeur calculée à partir d’autres, recalculée automatiquement | frontend/src/App.vue:14-15<br/>const vueCourante = computed(() => onglets.find((o) => o.id === ongletActif.value).composant) : c’est une valeur calculée à partir d’autres données, recalculée automatiquement quand celles-ci changent. |
| `v-for`                | répéter un élément pour chaque élément d’une liste                | frontend/src/App.vue:26-3<br/><button v-for="onglet in onglets" ...> : Vue répète le bloc pour chaque élément de la liste onglets.                                                                                                       |
| `v-if` / `v-else`      | afficher un élément seulement si une condition est vraie          | frontend/src/views/HistoriqueView.vue:23-49<br/> l’élément s’affiche seulement si la condition est vraie, sinon on affiche l’autre.                                                                                                      |
| `v-model`              | lier un champ de formulaire à une variable, dans les deux sens    | frontend/src/views/CommandeView.vue:72-75<br/> <input v-model.number="numeroTable" ... /> : le champ de formulaire et la variable numeroTable sont liés dans les deux sens.                                                              |
| `defineProps`          | recevoir des données du composant parent                          | frontend/src/components/PlatList.vue:6-8<br/> const props = defineProps({ plats: { type: Array, required: true } }) : le composant reçoit des données du parent via les props.                                                           |
| `defineEmits` / `emit` | envoyer un événement au composant parent                          | frontend/src/components/PlatList.vue:10-11 et 28-29<br/> const emit = defineEmits(['modifier', 'supprimer']) puis @click="emit('modifier', plat)" : le composant envoie un événement au parent pour lui demander une action.             |
| `onMounted`            | exécuter du code quand le composant apparaît                      | frontend/src/views/HistoriqueView.vue:10-12<br/> onMounted(async () => { commandes.value = await api.commandes.lister() }) : le code s’exécute quand le composant est affiché pour la première fois.                                     


## 9.
`Panier.vue` émet les événements `plus` et `moins` pour demander au composant parent de modifier les quantités. Le parent gère l’état global de la commande, garantissant ainsi la cohérence des données.

## 10.
Le `watch` dans `PlatForm.vue` surveille les changements de la prop `plat` et actualise les champs du formulaire pour afficher les données du plat sélectionné. Sans lui, les anciennes informations resteraient affichées.

## 11.
La liste déroulante de `PlatForm.vue` récupère les catégories via l’API `/api/categories`, définie dans `Routes.php` et gérée par `Categories.php`. Les données sont stockées dans une variable réactive pour afficher automatiquement les catégories dans le formulaire.