import socket
import threading
import json

HOST = '127.0.0.1'
PORT = 12345

BUFFER = ''
USERNAME = None
CHANNEL = "#général"

def listen(server_socket):
    global BUFFER
    while True:
        try:
            data = server_socket.recv(1024).decode()
            if not data:
                break
            BUFFER += data
            while '\n' in BUFFER:
                line, BUFFER = BUFFER.split('\n', 1)
                msg = json.loads(line)
                handle_message(msg)
        except Exception as e:
            print("[Erreur réception]:", e)
            break

def handle_message(msg):
    if msg['type'] == 'message':
        print(f"[{msg.get('channel', '#général')}] {msg['from']}: {msg['content']}")
    elif msg['type'] == 'private':
        print(f"[MP de {msg['from']}] {msg['content']}")
    elif msg['type'] == 'status':
        print(f"[Système] {msg['user']} est maintenant {msg['state']}")
    elif msg['type'] == 'system':
        print(f"[Système] {msg['content']}")
    elif msg['type'] == 'who':
        print(f"[Utilisateurs sur {msg['channel']}] : {', '.join(msg['users'])}")
    elif msg['type'] == 'error':
        print(f"[Erreur] {msg['message']}")
    else:
        print(f"[Inconnu] {msg}")

def send_message(server_socket, content):
    if content.startswith("/mp "):
        try:
            _, to, *msg = content.split()
            msg = ' '.join(msg)
            data = {
                "type": "private",
                "to": to,
                "content": msg
            }
            server_socket.sendall((json.dumps(data) + '\n').encode())
        except:
            print("[Usage] /mp <utilisateur> <message>")
    elif content.startswith("/join "):
        global CHANNEL
        CHANNEL = content.split(maxsplit=1)[1]
        print(f"🔁 Rejoint le canal {CHANNEL}")
    elif content.startswith("/who"):
        server_socket.sendall((json.dumps({"type": "who", "channel": CHANNEL}) + '\n').encode())
    elif content.startswith("/status "):
        status = content.split(maxsplit=1)[1]
        server_socket.sendall((json.dumps({"type": "status", "state": status}) + '\n').encode())
    else:
        data = {
            "type": "message",
            "channel": CHANNEL,
            "content": content
        }
        server_socket.sendall((json.dumps(data) + '\n').encode())

def main():
    global USERNAME
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
        server_socket.connect((HOST, PORT))
        threading.Thread(target=listen, args=(server_socket,), daemon=True).start()

        while True:
            if not USERNAME:
                print("Tapez /register ou /login")
                cmd = input("> ")
                if cmd.startswith("/register"):
                    _, username, password = cmd.split()
                    USERNAME = username
                    server_socket.sendall(json.dumps({"type": "register", "username": username, "password": password}).encode() + b'\n')
                elif cmd.startswith("/login"):
                    _, username, password = cmd.split()
                    USERNAME = username
                    server_socket.sendall(json.dumps({"type": "login", "username": username, "password": password}).encode() + b'\n')
            else:
                msg = input("> ")
                send_message(server_socket, msg)

if __name__ == '__main__':
    main()
