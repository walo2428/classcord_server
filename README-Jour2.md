
# 🧩 ClassCord Server - Déploiement Réseau (Jour 2)

## 📌 Objectif du jour
Rendre le serveur Python accessible sur le réseau local pour permettre aux clients SLAM de s’y connecter. Automatiser le lancement avec `systemd` et préparer une image Docker.

## 🖥️ Environnement de travail
- **OS** : Ubuntu Server 22.04 (VM VirtualBox)
- **Python** : 3.11.4
- **Réseau** : NAT avec redirection de port (port 12345)
- **Serveur cloné depuis** : https://github.com/AstrowareConception/classcord-server

## ✅ Étapes réalisées

### 1. Clonage et lancement local
```bash
git clone https://github.com/AstrowareConception/classcord-server.git
cd classcord-server
python3 server_classcord.py
```
Serveur accessible localement sur `localhost:12345`.

### 2. Vérification du port
```bash
sudo ufw allow 12345/tcp
ss -tulpn | grep 12345
```
Port bien ouvert pour écoute TCP.

### 3. Accès depuis une machine SLAM (via IP publique + redirection NAT)
- IP publique hôte : `10.0.108.XX`
- Port exposé : `12345`
- Connexion client réussie : ✅

### 4. Utilisateur système dédié
```bash
sudo useradd -m classcord
sudo passwd classcord
su - classcord
```

### 5. Service `systemd`
Fichier `/etc/systemd/system/classcord.service` :
```ini
[Unit]
Description=Serveur ClassCord
After=network.target

[Service]
User=classcord
WorkingDirectory=/home/classcord/classcord-server
ExecStart=/usr/bin/python3 /home/classcord/classcord-server/server_classcord.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Activation :
```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now classcord.service
```

### 6. Dockerisation
- Fichier `Dockerfile` créé à la racine du projet :
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt || true
EXPOSE 12345
CMD ["python", "server_classcord.py"]
```

- Construction et exécution :
```bash
docker build -t classcord-server .
docker run -it --rm -p 12345:12345 classcord-server
```

### 7. Documentation technique créée
- `DOC_CONNEXION.md` : Instructions de connexion depuis un poste client SLAM
- `README.md` : État du serveur et configuration réseau

## 🖼️ Captures d’écran (non incluses ici)
- Connexion d’un client invité
- Message reçu par le serveur
- `systemctl status classcord.service`

## 🧠 Compétences mobilisées
- Déploiement réseau local (NAT, firewall)
- Création de services `systemd`
- Dockerisation pour portabilité
- Documentation utilisateur et technique

---
📅 Dernière mise à jour : 2025-06-18
