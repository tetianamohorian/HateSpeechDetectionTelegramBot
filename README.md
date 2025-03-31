# Hate Speech Bot - Kubernetes Deployment

## 1. Opis čo robí Vaša aplikácia  
Táto aplikácia je chatbot pre platformu Telegram, ktorý detekuje a blokuje toxické správy (hate speech) v skupinách a kanáloch. Aplikácia reaguje na správy, vyhodnocuje ich obsah a ak deteguje neprípustný jazyk, automaticky odošle upozornenie a vymaže danú správu. Aplikácia používa MySQL databázu na ukladanie údajov o správcovských akciách a používateľoch, ktorí porušili pravidlá.  

---

## 2. Zoznam použitých kontajnerov a ich stručný opis  
- **tetianamohorian/hate-speech-bot:latest:** Tento kontajner obsahuje samotného bota, ktorý používa Python a knižnicu python-telegram-bot. Bot sleduje prichádzajúce správy a vyhodnocuje ich obsah.  
- **mysql:8:** Kontajner s MySQL databázou. Skladovanie dát ako je informácie o používateľoch a správy, ktoré sú považované za toxické.  

---

## 3. Zoznam Kubernetes objektov a ich stručný opis  
- **Namespace (botspace):** Vytvára samostatné prostredie v Kubernetes, kde sú umiestnené všetky objekty.  
- **PersistentVolume (mysql-pv):** Definuje trvalý zväzok pre MySQL databázu.  
- **PersistentVolumeClaim (mysql-pvc):** Požiadavka na trvalý zväzok pre MySQL.  
- **StatefulSet (mysql):** Kubernetes objekt pre nasadenie MySQL s jedným replikovaným podom.  
- **Deployment (bot-deployment):** Definuje nasadenie bota, zabezpečuje replikáciu a správu.  
- **Service (mysql):** Exponuje MySQL ako službu v Kubernetes, umožňuje pripojenie bota k databáze.  

---

## 4. Opis virtuálnych sietí a pomenovaných zväzkov ktoré aplikácia využíva  
### Virtuálne siete (Pod-to-Pod komunikácia)  
Aplikácia komunikuje cez internú sieť Kubernetes, kde sa služby ako MySQL a bot nachádzajú v rovnakom namespace (`botspace`), čo umožňuje priamu komunikáciu medzi podmi bez nutnosti otvárať externý prístup.  

### Pomenované zväzky  
- **PersistentVolume (mysql-pv):** Tento zväzok uchováva dáta MySQL, a to aj po reštartovaní podov.  
- **PersistentVolumeClaim (mysql-pvc):** Tento objekt vytvára požiadavku na trvalý zväzok, ktorý bude využívaný podom s MySQL databázou.  

---

## 5. Opis konfigurácie kontajnerov ktorú ste vykonali  
### Bot Container (`hate-speech-bot`)  
- **Obraz:** `tetianamohorian/hate-speech-bot:latest`  
- **Prostredie:** Nastavenie environment variables pre token bota (s použitím Kubernetes secret) a pripojenie na MySQL.  
- **Ports:** Kontajner neexponuje žiadne porty, pretože komunikácia s botom prebieha cez Telegram API.  

### MySQL Container  
- **Obraz:** `mysql:8`  
- **Prostredie:** Nastavenie root hesla pomocou environment variable `MYSQL_ROOT_PASSWORD`.  
- **Volumes:** Mapped to persistent volume `mysql-pv` a `mysql-pvc` na uchovávanie databázových súborov.  

---

## 6. Návod ako pripraviť, spustiť, pozastaviť a vymazať aplikáciu  

### Príprava aplikácie:  
1. Skontrolujte, či máte správne nainštalovaný Docker a Kubernetes.  
2. Build Docker obraz pre bota:  
```bash
docker build -t tetianamohorian/hate-speech-bot:latest .
```
3. Push obraz na Docker Hub:
```bash
docker push tetianamohorian/hate-speech-bot:latest
```

4. Spustenie aplikácie:
Vytvorte Kubernetes namespace (ak ešte neexistuje):

```bash
kubectl create namespace botspace
```

5. Spustite aplikáciu:

```bash
./start-app.sh
```

6. Pozastavenie aplikácie:
Pozastavte pod aplikácie:

```bash
kubectl delete pod -n botspace -l app=bot
```

7. Vymazanie aplikácie:
Vymažte všetky objekty v Kubernetes:

```bash
./stop-app.sh
```

## 7. Návod ako si pozrieť aplikáciu na webovom prehliadači
Aplikácia je navrhnutá tak, aby fungovala cez Telegram API. Nemá webový front-end. Ak chcete pozerať aktivitu bota, môžete si pozrieť správy z Telegramu priamo v aplikácii Telegram a komunikovať s botom prostredníctvom Telegram klienta.

Zobrazenie logov aplikácie:
Logy bota:

```bash
kubectl logs -n botspace -l app=bot
```

Logy MySQL:

```bash
kubectl logs mysql-0 -n botspace
```
