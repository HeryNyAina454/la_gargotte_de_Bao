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