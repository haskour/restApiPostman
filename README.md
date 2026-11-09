# Rapport de TP : Validation et Analyse des Scénarios d'Erreurs d'une API REST sous Postman

## 1. Contexte et Objectifs du TP
L'objectif de ce TP est de concevoir et d'exécuter une suite de tests automatisés avec **Postman** pour valider le comportement d'une API REST (DummyJSON Products et Users). 

Une attention particulière est portée à la gestion standardisée des erreurs HTTP (notamment les codes `400 Bad Request` et `404 Not Found`), à l'extraction dynamique de variables d'environnement et aux mesures de temps de réponse.

---

## 2. Synthèse des Observations et Explication des Codes HTTP

| Code HTTP | Libellé | Usage dans le TP & Description |
| :--- | :--- | :--- |
| **200** | OK | Requêtes `GET` (recherche, catégories, produit par ID) et modifications (`PUT`, `PATCH`). Indique que la requête a été traitée avec succès par le serveur et que les données demandées sont renvoyées. |
| **201** | Created | Requêtes `POST` (`AddProduct`, `CreateUser`). Confirme la création effective d'une nouvelle ressource sur le serveur. |
| **400** | Bad Request | Envoi de données malformées ou de types d'attributs invalides dans le corps JSON (ex: du texte à la place d'un nombre pour un prix). Indique un rejet par l'API dû à une erreur côté client. |
| **404** | Not Found | Interrogation d'un produit avec un ID inexistant (`/products/99999`). Signale que le serveur ne trouve aucune ressource correspondant à l'URI fournie. |

---

## 3. Détail des Requêtes et des Tests Automatisés

### 3.1. Dossier : Création

#### A. Requête `AddProduct` (`POST {{baseUrl}}/products/add`)
* **Description :** Ajout d'un produit (*Casque Audio Sans Fil*, `price`: 129.99, `category`: "audio", `stock`: 25).
* **Script de Test Postman :**
```javascript
// Vérifier que le statut de création est 201 ou 200
pm.test("Statut de création valide", function () {
    pm.response.to.have.status(201);
});

// Extraire l'ID du produit créé et le stocker dans les variables d'environnement
var jsonData = pm.response.json();
pm.environment.set("createdProductId", jsonData.id);
console.log("ID du produit créé : " + jsonData.id);
```

<img width="1422" height="985" alt="Screenshot From 2026-11-09 00-22-26" src="https://github.com/user-attachments/assets/7a787fbd-f0ea-479d-8777-191d68c1f4c2" />


---

#### B. Requête `CreateUser` (`POST {{baseUrl}}/users/add`)
* **Description :** Création d'un profil utilisateur (*Hicham Askour*).
* **Script de Test Postman :**
```javascript
pm.test("Vérifier que le temps de réponse (Response Time) est inférieur à 500 ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});
```

<img width="1422" height="985" alt="Screenshot From 2026-11-09 00-23-30" src="https://github.com/user-attachments/assets/19723ead-01fd-454b-965e-372411bfaa1b" />

---

### 3.2. Dossier : Lecture et Scénarios d'Erreurs

#### A. Consultation et Test d'Erreur `404 Not Found` (`GET {{baseUrl}}/products/99999`)
* **Description :** Interrogation d'un produit avec un identifiant inexistant.
* **Script de Test Postman :**
```javascript
pm.test("Le code de statut est 404 Not Found", function () {
    pm.response.to.have.status(404);
});

pm.test("Le corps de la réponse contient un message d'erreur clair", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('message');
    pm.expect(jsonData.message).to.include("not found");
});
```

* **Exemple de réponse JSON standardisée obtenue :**
```json
{
  "message": "Product with id '99999' not found"
}
```

<img width="1422" height="985" alt="Screenshot From 2026-11-09 00-25-37" src="https://github.com/user-attachments/assets/b6e9e9ee-f06a-474b-9e9b-0c7d6086c299" />

---

#### B. Validation du Cas d'Erreur `400 Bad Request`
* **Description :** Soumission d'un payload JSON non valide ou comportant des types incorrects (ex: champ `price` contenant la valeur `"abc"` au lieu d'un nombre).
* **Script de Test Postman :**
```javascript
pm.test("Le code de statut est 400 Bad Request", function () {
    pm.response.to.have.status(400);
});

pm.test("La réponse indique une erreur de validation ou de format", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.satisfy(function(body) {
        return body.message !== undefined || body.error !== undefined;
    });
});

### 3.3. Dossier : Modification et Suppression

* **`EditProduct` (`PUT`) & `PatchProduct` (`PATCH`) :** Mises à jour totales et partielles du produit via l'ID réutilisé (`{{productId}}`).

<img width="1422" height="985" alt="Screenshot From 2026-11-09 00-28-47" src="https://github.com/user-attachments/assets/7f678177-1cc5-4839-a7a5-481bfc71bc1e" />

* **`DeleteProduct` (`DELETE {{baseUrl}}/products/1`) :** Validation de la suppression de la ressource.
<img width="1422" height="985" alt="Screenshot From 2026-11-09 00-28-05" src="https://github.com/user-attachments/assets/e82b8ffd-eaed-425a-b370-abe302a48bc9" />

> **[Insérer la capture d'écran Postman - Exécution globale dans le Collection Runner]**

---

## 4. Conclusion
Les tests automatisés exécutés sous Postman démontrent l'importance d'accompagner chaque code de statut HTTP d'un message d'erreur explicite au format JSON (comme les erreurs `400` et `404`). L'utilisation des scripts d'assertion `pm.test` garantit la fiabilité des API REST et facilite leur intégration future dans un pipeline d'intégration continue (CI/CD).
