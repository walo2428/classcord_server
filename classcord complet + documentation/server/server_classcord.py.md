import socket
import threading
import json
import pickle
import os
import logging
from datetime import datetime

# === Configuration des logs ===
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler("server.log"),
        logging.StreamHandler()
    ]
)

HOST = '0.0.0.0'
PORT = 12345

USER_FILE = 'users.pkl'
CLIENTS = {}     # socket: username
USERS = {}       # username: password
CHANNELS = {}    # channel_name: list of sockets
LOCK = threading.Lock()


def load_users():
    global USERS
    if os.path.exists(USER_FILE):
        with open(USER_FILE, 'rb') as f:
            USERS = pickle.load(f)
    logging.info(f"[INIT] Utilisateurs chargés: {list(USERS.keys())}")


def save_users():
    with open(USER_FILE, 'wb') as f:
        pickle.dump(USERS, f)
    logging.info("[SAVE] Utilisateurs sauvegardés.")


def send_system_message(to, content):
    message = {
        "type": "system",
        "from": "server",
        "timestamp": datetime.now().isoformat(),
        "content": content
    }
    try:
        to.sendall((json.dumps(message) + '\n').encode())
    except Exception as e:
        logging.error(f"Erreur envoi message système à {CLIENTS.get(to)} : {e}")


def broadcast(message, sender_socket=None):
    channel = message.get("channel", "#général")
    if channel not in CHANNELS:
        return
    for client_socket in CHANNELS[channel]:
        if client_socket != sender_socket:
            try:
                client_socket.sendall((json.dumps(message) + '\n').encode())
                logging.info(f"Message envoyé à {CLIENTS.get(client_socket)} : {message}")
            except Exception as e:
                logging.error(f"Échec d'envoi à {CLIENTS.get(client_socket)} : {e}")


def handle_client(client_socket):
    buffer = ''
    username = None
    address = client_socket.getpeername()
    logging.info(f"[CONNEXION] Nouvelle connexion depuis {address}")

    try:
        while True:
            data = client_socket.recv(1024).decode()
            if not data:
                break
            buffer += data
            while '\n' in buffer:
                line, buffer = buffer.split('\n', 1)
                logging.info(f"[RECU] {address} >> {line}")
                msg = json.loads(line)

                if msg['type'] == 'register':
                    with LOCK:
                        if msg['username'] in USERS:
                            response = {'type': 'error', 'message': 'Username already exists.'}
                        else:
                            USERS[msg['username']] = msg['password']
                            save_users()
                            response = {'type': 'register', 'status': 'ok'}
                        client_socket.sendall((json.dumps(response) + '\n').encode())

                elif msg['type'] == 'login':
                    with LOCK:
                        if USERS.get(msg['username']) == msg['password']:
                            username = msg['username']
                            CLIENTS[client_socket] = username
                            response = {'type': 'login', 'status': 'ok', 'username': username}
                            client_socket.sendall((json.dumps(response) + '\n').encode())
                            logging.info(f"[LOGIN] {username} connecté")

                            # Ajouter automatiquement au canal #général
                            if "#général" not in CHANNELS:
                                CHANNELS["#général"] = []
                            if client_socket not in CHANNELS["#général"]:
                                CHANNELS["#général"].append(client_socket)
                                logging.info(f"[CANAL] {username} rejoint #général automatiquement")

                            # Informer les autres utilisateurs
                            broadcast({'type': 'status', 'user': username, 'state': 'online'}, client_socket)
                        else:
                            response = {'type': 'error', 'message': 'Login failed.'}
                            client_socket.sendall((json.dumps(response) + '\n').encode())

                elif msg['type'] == 'message':
                    if not username:
                        username = msg.get('from', 'invité')
                        with LOCK:
                            CLIENTS[client_socket] = username
                        logging.info(f"[INFO] Connexion invitée détectée : {username}")

                    msg['from'] = username
                    msg['timestamp'] = datetime.now().isoformat()
                    channel = msg.get('channel', '#général')

                    with LOCK:
                        if channel not in CHANNELS:
                            CHANNELS[channel] = []
                        if client_socket not in CHANNELS[channel]:
                            CHANNELS[channel].append(client_socket)

                    if msg.get('subtype') == 'private':
                        dest_user = msg.get('to')
                        with LOCK:
                            target_socket = next((s for s, u in CLIENTS.items() if u == dest_user), None)
                        if target_socket:
                            try:
                                target_socket.sendall((json.dumps(msg) + '\n').encode())
                                logging.info(f"[MP] {username} >> {dest_user} : {msg['content']}")
                            except Exception as e:
                                logging.error(f"[ERREUR] Envoi MP à {dest_user} : {e}")
                        else:
                            send_system_message(client_socket, f"L'utilisateur {dest_user} n'est pas connecté.")
                    else:
                        logging.info(f"[{channel}] {username} >> {msg['content']}")
                        broadcast(msg, client_socket)

                elif msg['type'] == 'status' and username:
                    broadcast({'type': 'status', 'user': username, 'state': msg['state']}, client_socket)
                    logging.info(f"[STATUS] {username} est maintenant {msg['state']}")

                elif msg['type'] == 'users':
                    with LOCK:
                        connected_users = list(CLIENTS.values())
                    response = {'type': 'users', 'users': connected_users}
                    client_socket.sendall((json.dumps(response) + '\n').encode())
                    logging.info(f"[INFO] Liste des utilisateurs envoyée à {username or address}")

    except Exception as e:
        logging.error(f"[ERREUR] Problème avec {address} ({username}): {e}")
    finally:
        if username:
            broadcast({'type': 'status', 'user': username, 'state': 'offline'}, client_socket)
        with LOCK:
            CLIENTS.pop(client_socket, None)
            for ch in CHANNELS.values():
                if client_socket in ch:
                    ch.remove(client_socket)
        client_socket.close()
        logging.info(f"[DECONNEXION] {address} déconnecté")


def admin_interface():
    print("Interface admin démarrée")
    while True:
        print("\n[Interface d'administration]")
        print("1. Voir les clients connectés")
        print("2. Envoyer un message système")
        print("3. Fermer un canal")
        print("4. Quitter")
        choix = input("> ")

        if choix == "1":
            with LOCK:
                for sock, user in CLIENTS.items():
                    print(f" - {user} ({sock.getpeername()})")
        elif choix == "2":
            contenu = input("Contenu du message système : ")
            with LOCK:
                for sock in CLIENTS:
                    send_system_message(sock, contenu)
        elif choix == "3":
            canal = input("Nom du canal à fermer (ex: #dev): ")
            with LOCK:
                if canal in CHANNELS:
                    for sock in CHANNELS[canal]:
                        send_system_message(sock, f"Le canal {canal} a été fermé par l'admin.")
                    del CHANNELS[canal]
                    print(f"✅ Canal {canal} fermé.")
                else:
                    print("Canal introuvable.")
        elif choix == "4":
            print("Arrêt du serveur...")
            os._exit(0)


def main():
    load_users()
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((HOST, PORT))
    server_socket.listen()
    logging.info(f"Serveur en écoute sur {HOST}:{PORT}")

    if os.getenv("ADMIN", "false").lower() == "true":
        threading.Thread(target=admin_interface, daemon=True).start()

    while True:
        client_socket, addr = server_socket.accept()
        threading.Thread(target=handle_client, args=(client_socket,), daemon=True).start()


if __name__ == '__main__':
    main()
