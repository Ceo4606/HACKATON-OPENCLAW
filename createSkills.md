---

### 1. Configuration du "Master Provider" (Groq)

Dans les réglages (Settings/Models) de ton interface OpenClaw :

* **Provider :** Choisis `OpenAI` (car Groq utilise le même format).
* **Base URL :** `[https://api.groq.com/openai/v1](https://api.groq.com/openai/v1)`
* **API Key :** Ta clé Groq.
* **Model :** `llama3-70b-8192` (ou `llama3-8b-8192` pour une vitesse foudroyante).

---

### 2. Création des Skills (La logique métier)

Dans l'onglet **Skills**, tu vas créer les "capacités" de ton bot. Voici les structures à copier/coller :

#### A. Skill : `check_payment` (Vérification Mobile Money)

Ce skill permet à l'IA d'aller demander à Laravel si l'argent est arrivé.

* **Name :** `check_payment`
* **Description :** "Vérifie si le client a payé via Mobile Money."
* **Action (HTTP) :**
* **Method :** `GET`
* **URL :** `http://[TON_IP_VPS]:8000/api/payments/verify/{{vendeur_id}}`


* **Parameters :** `customer_phone` (string).

#### B. Skill : `generate_image` (Branding Pro)

* **Name :** `improve_image`
* **Description :** "Prend une photo brute et génère une version professionnelle."
* **Action (HTTP) :**
* **Method :** `POST`
* **URL :** `http://[TON_IP_VPS]:8000/api/media/beautify`


* **Parameters :** `image_url` (string).

---

### 3. Le "System Prompt" Master (L'identité de Lobi-Bot)

Dans la configuration de ton Agent principal, tu dois définir ses instructions. C’est ici qu’on définit le comportement pour Kinshasa.

**Copie ceci dans le "System Instructions" :**

> "Tu es Lobi-Bot, l'assistant intelligent des commerçants de Kinshasa.
> 1. Ton rôle est d'aider les vendeurs à gérer leurs stocks et les clients à acheter.
> 2. Tu parles un mélange de Français et de Lingala poli ('Mboté', 'Merci mingi').
> 3. Si un client veut payer, utilise le skill `check_payment`.
> 4. Si un vendeur envoie une photo avec le mot 'publier', utilise `improve_image` puis `publier_produit`.
> 5. Sois concis, les gens n'aiment pas lire de longs textes sur WhatsApp."
> 
> 

---

### 4. Gestion du Multi-Vendeur (Dynamic Context)

Puisque tu es dans l'interface web, pour que l'IA sache **quel** vendeur elle aide, tu dois configurer les **Variables de Session**.

Dans l'onglet **Context** ou **Variables** de l'agent :

* Ajoute une variable `vendeur_id`.
* Lorsqu'un message arrive, OpenClaw doit extraire le numéro de destination (le téléphone du vendeur) pour le mapper à cet ID dans ton Laravel.

---

### 5. Préparation de la commande `/publier_produits`

Dans l'interface, crée un **Command Trigger** ou un **Pattern Matcher** :

* **Trigger :** `/publier`
* **Action :** Déclenche un flux qui :
1. Récupère le dernier produit ajouté en DB via Laravel.
2. Demande à l'IA de rédiger une légende accrocheuse.
3. Utilise le connecteur WhatsApp pour "Post Status".



---

### 🚀 Pourquoi faire ça dans l'interface maintenant ?

* **Visualisation :** Tu vois immédiatement si tes skills sont bien enregistrés.
* **Test direct :** Tu as souvent une fenêtre de "Chat Test" sur le côté de l'interface OpenClaw. Tu peux simuler une réponse de Laravel (en JSON) pour voir comment l'IA réagit.

### La prochaine étape pour Sam & Ilunga :

Une fois que ces "coquilles vides" (les skills) sont créées dans l'interface :

1. **Sam :** Tu codes les routes correspondantes dans Laravel (`routes/api.php`).
2. **Ilunga :** Tu connectes le téléphone de test au QR code affiché dans l'interface.

**Tu veux que je te donne le format JSON exact que ton API Laravel doit renvoyer pour que le skill `check_payment` soit compris par l'IA d'OpenClaw ?** (Si le JSON est mal structuré, l'IA va bugger).
