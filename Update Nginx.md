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

Dacă vrei, pot să fac o **versiune ultra-condensată**, cu toate comenzile într-un **script ready-to-use**, astfel încât să rulezi totul cu 5-6 comenzi fără să mai cauți manual.

Vrei să fac asta?
