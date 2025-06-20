import sqlite3
import csv
import json
import sys

DB_NAME = 'classcord.db'

def fetch_all_messages():
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute("SELECT sender, content, timestamp, channel FROM messages ORDER BY timestamp ASC")
    messages = c.fetchall()
    conn.close()
    return messages

def export_csv(filename="messages.csv"):
    messages = fetch_all_messages()
    with open(filename, 'w', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow(["sender", "content", "timestamp", "channel"])
        writer.writerows(messages)
    print(f"[✔] Messages exportés dans {filename}")

def export_json(filename="messages.json"):
    messages = fetch_all_messages()
    message_list = [
        {
            "sender": m[0],
            "content": m[1],
            "timestamp": m[2],
            "channel": m[3]
        } for m in messages
    ]
    with open(filename, 'w', encoding='utf-8') as f:
        json.dump(message_list, f, indent=4, ensure_ascii=False)
    print(f"[✔] Messages exportés dans {filename}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage : python3 export_messages.py [csv|json]")
        sys.exit(1)

    format_type = sys.argv[1].lower()
    if format_type == "csv":
        export_csv()
    elif format_type == "json":
        export_json()
    else:
        print("Format non reconnu. Utilise : csv ou json.")