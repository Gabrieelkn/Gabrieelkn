Absolut! 💪 Hai să-ți fac un **ghid complet pas cu pas și cu toate comenzile utile** pentru viitoarele modificări ale aplicației și bazei de date, astfel încât să poți actualiza rapid `SSMApi.dll` și Nginx fără să pierzi timp sau să blochezi portul.

---

# 📌 Ghid complet pentru actualizări API + baza de date

---

## **1️⃣ Oprește aplicația / serviciul**

Dacă rulezi prin **systemd**:

```bash
sudo systemctl stop api.service
```

Dacă rulezi manual (dotnet):

```bash
ps aux | grep SSMApi.dll
sudo kill <PID>
```

> Dacă procesul e blocat, folosește `sudo kill -9 <PID>`.

Verifică că portul 5000 e liber:

```bash
sudo lsof -i :5000
```

---

## **2️⃣ Back-up (opțional dar recomandat)**

### a) Backup aplicație:

```bash
cp -r /home/gabi/publish /home/gabi/publish_backup_$(date +%F_%H%M)
```

### b) Backup baza de date (exemplu MySQL):

```bash
mysqldump -u <user> -p<parola> <database> > /home/gabi/db_backup_$(date +%F_%H%M).sql
```

---

## **3️⃣ Șterge / înlocuiește aplicația**

### a) Ștergere folder vechi:

```bash
sudo rm -rf /home/gabi/publish
```

### b) Creează folder nou:

```bash
mkdir -p /home/gabi/publish
sudo chown -R gabi:gabi /home/gabi/publish
```

### c) Copiază fișierele noi

De exemplu, dintr-un ZIP sau transfer SCP/rsync:

```bash
scp user@pc_local:/cale/SSMApi.dll /home/gabi/publish/
scp -r user@pc_local:/cale/config/* /home/gabi/publish/
```

---

## **4️⃣ Testare manuală**

```bash
cd /home/gabi/publish
dotnet SSMApi.dll
```

* Ar trebui să vezi:

```
Now listening on: http://0.0.0.0:5000
Application started. Press Ctrl+C to shut down.
```

Test rapid backend:

```bash
curl http://127.0.0.1:5000
```

---

## **5️⃣ Actualizare serviciu systemd**

Editează fișierul `api.service` dacă e necesar:

```bash
sudo nano /etc/systemd/system/api.service
```

* `ExecStart` → `/home/gabi/publish/SSMApi.dll`
* `WorkingDirectory` → `/home/gabi/publish`

Reîncarcă systemd:

```bash
sudo systemctl daemon-reload
sudo systemctl restart api.service
sudo systemctl status api.service
```

Loguri live:

```bash
sudo journalctl -u api.service -f
```

---

## **6️⃣ Verificare prin Nginx**

Test direct port local:

```bash
curl http://127.0.0.1:5000
```

Test prin reverse proxy HTTPS:

```bash
curl -vk https://api.ssmbucuresti.ro
```

Reîncarcă Nginx dacă ai modificat config:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## **7️⃣ Migrarea bazei de date (dacă ai modificări)**

### a) Aplica migrarea (Entity Framework Core):

```bash
cd /home/gabi/publish
dotnet ef database update
```

### b) Sau restaurare backup:

```bash
mysql -u <user> -p<parola> <database> < /home/gabi/db_backup.sql
```

---

## **8️⃣ Test final**

1. Accesează API-ul prin browser sau curl:

```bash
curl -vk https://api.ssmbucuresti.ro
```

2. Verifică funcționalitatea aplicației și logurile systemd:

```bash
sudo journalctl -u api.service -f
```

---
Perfect! 💪 Am creat un **script complet gata de folosit** pentru actualizarea aplicației și a bazei de date. Acest script presupune că ai fișierele noi pregătite și vrei să faci totul curat, fără să blochezi portul și cu servicii systemd/Nginx actualizate.

---

## **Script complet de update SSMApi.dll**

Salvează-l, de exemplu, ca `/home/gabi/update_api.sh` și fă-l executabil:

```bash
nano /home/gabi/update_api.sh
chmod +x /home/gabi/update_api.sh
```

Lipește următorul conținut:

```bash
#!/bin/bash
# ============================================
# Script de update pentru SSMApi.dll + DB
# ============================================

APP_DIR="/home/gabi/publish"
BACKUP_DIR="/home/gabi/publish_backup_$(date +%F_%H%M)"
SERVICE_NAME="api.service"
NGINX_HOST="api.ssmbucuresti.ro"

echo "1️⃣ Oprește serviciul systemd dacă rulează..."
sudo systemctl stop $SERVICE_NAME

echo "2️⃣ Oprește orice instanță manuală..."
sudo pkill -f SSMApi.dll

echo "3️⃣ Backup folder aplicație..."
if [ -d "$APP_DIR" ]; then
    cp -r "$APP_DIR" "$BACKUP_DIR"
    echo "Backup creat: $BACKUP_DIR"
fi

echo "4️⃣ Șterge folderul vechi..."
sudo rm -rf "$APP_DIR"

echo "5️⃣ Creează folder gol..."
mkdir -p "$APP_DIR"
sudo chown -R gabi:gabi "$APP_DIR"

echo "6️⃣ Copiază fișierele noi (modifică sursa după nevoie)..."
# Exemplu: rsync dintr-un folder local sau server
# rsync -avz /cale/catre/fisiere_noi/* "$APP_DIR/"

echo "7️⃣ Pornește aplicația manual pentru test..."
cd "$APP_DIR"
dotnet SSMApi.dll &
sleep 5

echo "8️⃣ Test backend..."
curl http://127.0.0.1:5000 || echo "❌ Backend nu răspunde"

echo "9️⃣ Update systemd..."
sudo systemctl daemon-reload
sudo systemctl restart $SERVICE_NAME
sudo systemctl status $SERVICE_NAME --no-pager

echo "🔟 Test Nginx..."
curl -vk https://$NGINX_HOST || echo "❌ Nginx nu redirecționează corect"

echo "✅ Update finalizat!"
```

---

### 🔹 Cum funcționează

1. Oprește serviciul systemd și orice instanță manuală.
2. Face backup al folderului `publish`.
3. Șterge vechiul folder și creează unul nou.
4. Copiază fișierele noi (poți modifica linia cu `rsync` sau `scp`).
5. Pornește aplicația manual pentru test rapid.
6. Testează backend-ul și Nginx.
7. Reîncarcă systemd și repornește serviciul.

---

### 🔹 Recomandări

* Înlocuiește linia cu `rsync` cu metoda ta de upload a fișierelor noi.
* Poți adăuga pas pentru **migrare DB** dacă folosești EF Core:

```bash
dotnet ef database update
```

* Poți rula script-ul de fiecare dată când faci update, fără să te mai stresezi cu portul sau serviciul blocat.

---
