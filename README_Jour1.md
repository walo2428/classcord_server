
#  Documentation — Jour 1

## Jour 1
Lundi 17 juin 2025

##  Étudiant
Nom : Omar Talibi  
Machine : VM Debian  
Utilisateur : projet

---

## Objectifs de la journée

- Préparer la machine virtuelle avec Debian
- Accéder et configurer le projet Python (serveur)
- Tester l’accès réseau depuis l’extérieur
- Mettre en place un environnement de test fonctionnel

---

## Étapes réalisées

### 1. Configuration de l’environnement
- Installation de la VM Debian sous VirtualBox
- Création de l'utilisateur `projet`
- Ajout de l'utilisateur `projet` au groupe `sudo` via `visudo`

### 2. Réseau
- Changement du mode NAT → mode **pont (bridge)** pour une vraie IP locale
- Récupération de l’adresse IP avec `ip a` :  
  ➤ `inet 10.0.108.113/24`

### 3. Projet Python
- Clonage du dépôt du projet
- Lancement du serveur avec :  
  ➤ `python3 server.py`  
  (Serveur en écoute sur `host = 0.0.0.0`, `port = 12345`)
- Test de connexion via `telnet` depuis une autre machine : **OK**

### 4. Sécurité & pare-feu
- Vérification de `ufw` :  
  ➤ `ufw status` → `inactive` (aucun port bloqué)

---

## Problèmes rencontrés

- `sudo` ne fonctionnait pas au départ :  
  ➤ corrigé avec ajout de `projet` dans les sudoers
- IP partagée entre plusieurs machines en NAT :  
  ➤ corrigé avec le mode pont
- Problème de connexion :  
  ➤ resolu en changeant le port manuelemnt `hote :12345`

---

## Informations techniques

- OS : Debian 11 (VM)
- Adresse IP locale : `10.0.108.113`
- Port d'écoute : `12345`
- Firewall : `ufw inactive`
