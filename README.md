---

# 🚀 Lobi-Bot : L'IA qui transforme WhatsApp en Boutique Intelligente

**Lobi-Bot** est une plateforme SaaS permettant aux commerçants de Kinshasa d'automatiser leur business via WhatsApp. Grâce à l'orchestration d'agents intelligents, le vendeur délègue la gestion des stocks, la vente et la création de contenu à une IA tout en gardant le contrôle humain.

---

## 🏗️ Architecture du Système

Le projet repose sur un "covoiturage" technologique optimisé pour un VPS (8GB RAM) :

* **Backend (Laravel 10 + Sail) :** Le cerveau logique. Il gère la base de données (SQLite), l'authentification, le catalogue produits et la vérification des paiements.
* **Orchestrateur (OpenClaw) :** Le pont entre WhatsApp et notre logique. Il transforme les messages textuels/vocaux en actions concrètes.
* **Intelligence Artificielle :** **Groq (Llama 3)** pour une vitesse de réponse instantanée.

---

## 📋 Tableau des Fonctionnalités & Articulation

| Fonctionnalité | Implémentation Backend (Sam) | Implémentation OpenClaw (Ilunga) |
| --- | --- | --- |
| **Chatbot Boutique** | API `/api/v1/context/{vendor_id}` | Agent avec System Prompt dynamique. |
| **Paiement Verify** | Liaison API Mobile Money (Orange/M-Pesa) | Skill `/verifier` qui interroge le statut SQL. |
| **Gestion Stock** | CRUD JSON (Produits, Prix, Quantité) | Skill `/ajouter` (Parse le texte vers l'API). |
| **Image Pro ✨** | Stockage Cloudinary / S3 | Skill `/generer_image` (Flux ou DALL-E). |
| **Status Update** | Générateur de liens de partage | Skill `/publier` (Injection via Gateway WA). |

---

## 🛠️ Guide d'Implémentation pour Ilunga (Débutant OpenClaw)

Ilunga, ton rôle est de créer les **Skills**. Un Skill est une "super-pouvoir" que tu donnes à l'IA pour qu'elle puisse parler à notre Backend Laravel.

### 1. Structure d'un Skill (Exemple : `/ajouter_produit`)

Dans le dossier `skills/` d'OpenClaw, crée un fichier `ajouter_produit.yaml` :

```yaml
name: ajouter_produit
description: "Permet au vendeur d'ajouter un produit au stock"
parameters:
  type: object
  properties:
    nom: {type: string, description: "Nom du produit"}
    prix: {type: number, description: "Prix en CDF ou USD"}
  required: [nom, prix]
endpoint:
  url: "http://lobi_backend/api/products" # Nom du service Docker de Sam
  method: POST
  headers:
    Authorization: "Bearer {{vendor_token}}"

```

### 2. Le System Prompt (Le rôle de l'Agent)

Tu dois définir comment l'IA se comporte.

> *"Tu es l'assistant de [NOM_BOUTIQUE]. Si le vendeur t'envoie une photo avec le mot 'publier', utilise le skill 'generer_image' puis 'publier_produit'. Réponds toujours de manière polie, de préférence en Lingala ou Français selon le client."*

---

## 🐳 Déploiement Docker (Covoiturage)

Nous utilisons un seul réseau Docker (`lobinet`) pour que les conteneurs se parlent sans passer par internet.

```yaml
# docker-compose.yml (Simplifié)
services:
  lobi_backend: # Backend de Sam
    build: ./backend
    container_name: lobi_backend
    networks: [- lobinet]

  openclaw: # Orchestrateur d'Ilunga
    image: openclaw/openclaw
    container_name: openclaw_master
    depends_on: [lobi_backend]
    networks: [- lobinet]

networks:
  lobinet:
    driver: bridge

```

---

## 📸 Focus : Nouvelles Fonctionnalités "Elite"

### `/generer_image`

* **Problème :** Les vendeurs ont souvent des caméras de mauvaise qualité ou un arrière-plan encombré.
* **Solution :** Le vendeur envoie une photo. OpenClaw appelle une API d'IA générative. Le backend reçoit l'image "Studio" et met à jour le catalogue.

### `/publier_produits`

* **Action :** Une commande unique pour déclencher une multi-publication.
* **Flux :** IA -> Laravel (récupère le lien) -> OpenClaw (envoie vers Status WhatsApp + Groupes de vente).

---

## 👨‍💻 Workflow de Travail à Distance

1. **Code Local :** Sam travaille sur Laravel en local.
2. **Tunneling :** `ssh -R 8000:localhost:8000` pour que l'OpenClaw du VPS puisse tester le code de Sam en direct.
3. **UI de contrôle :** Accès via `ssh -L 3000:localhost:54962` pour scanner les QR codes de test.

---

*Lobi-Bot - Hackathon 2024 - "L'avenir du commerce est dans votre poche."*
