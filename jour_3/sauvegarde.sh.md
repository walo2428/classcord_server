#!/bin/bash

# Dossier de destination
DEST=~/BTS_SIO/classcord-server/backups

# Fichier source à sauvegarder
SOURCE=~/BTS_SIO/classcord-server/users.pkl

# Création du dossier de sauvegarde si inexistant
mkdir -p "$DEST"

# Sauvegarde avec horodatage
cp "$SOURCE" "$DEST/users_$(date +%Y%m%d_%H%M%S).pkl"