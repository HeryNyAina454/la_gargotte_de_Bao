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
        