  
## PARTE 1: GUARDAR TODOS LOS EQUIPOS COMO HASHES  
  
### AFC EAST (4 equipos)  
```redis  
HSET team:1 teamId 1 name "Buffalo Bills" alias "BUF" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/buf.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "EAST"  
  
HSET team:2 teamId 2 name "Miami Dolphins" alias "MIA" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/mia.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "EAST"  
  
HSET team:3 teamId 3 name "New England Patriots" alias "NE" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/ne.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "EAST"  
  
HSET team:4 teamId 4 name "New York Jets" alias "NYJ" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/nyj.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "EAST"  
```  
  
### AFC NORTH (4 equipos)  
```redis  
HSET team:5 teamId 5 name "Baltimore Ravens" alias "BAL" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/bal.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "NORTH"  
  
HSET team:6 teamId 6 name "Cincinnati Bengals" alias "CIN" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/cin.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "NORTH"  
  
HSET team:7 teamId 7 name "Cleveland Browns" alias "CLE" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/cle.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "NORTH"  
  
HSET team:8 teamId 8 name "Pittsburgh Steelers" alias "PIT" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/pit.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "NORTH"  
```  
  
### AFC WEST (4 equipos)  
```redis  
HSET team:9 teamId 9 name "Denver Broncos" alias "DEN" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/den.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "WEST"  
  
HSET team:10 teamId 10 name "Kansas City Chiefs" alias "KC" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/kc.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "WEST"  
  
HSET team:11 teamId 11 name "Las Vegas Raiders" alias "LV" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/lv.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "WEST"  
  
HSET team:12 teamId 12 name "Los Angeles Chargers" alias "LAC" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/lac.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "WEST"  
```  
  
### AFC SOUTH (4 equipos)  
```redis  
HSET team:13 teamId 13 name "Houston Texans" alias "HOU" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/hou.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "SOUTH"  
  
HSET team:14 teamId 14 name "Indianapolis Colts" alias "JAC" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/jax.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "SOUTH"  
  
HSET team:15 teamId 15 name "Jacksonville Jaguars" alias "IND" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/ind.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "SOUTH"  
  
HSET team:16 teamId 16 name "Tennessee Titans" alias "TEN" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/ten.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "AFC" division "SOUTH"  
```  
  
### NFC EAST (4 equipos)  
```redis  
HSET team:17 teamId 17 name "Dallas Cowboys" alias "DAL" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/dal.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "EAST"  
  
HSET team:18 teamId 18 name "New York Giants" alias "NYG" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/nyg.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "EAST"  
  
HSET team:19 teamId 19 name "Philadelphia Eagles" alias "PHI" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/phi.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "EAST"  
  
HSET team:20 teamId 20 name "Washington Commanders" alias "WAS" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/wsh.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "EAST"  
```  
  
### NFC NORTH (4 equipos)  
```redis  
HSET team:21 teamId 21 name "Chicago Bears" alias "CHI" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/chi.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "NORTH"  
  
HSET team:22 teamId 22 name "Detroit Lions" alias "DET" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/det.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "NORTH"  
  
HSET team:23 teamId 23 name "Green Bay Packers" alias "GB" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/gb.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "NORTH"  
  
HSET team:24 teamId 24 name "Minnesota Vikings" alias "MIN" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/min.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "NORTH"  
```  
  
### NFC WEST (4 equipos)  
```redis  
HSET team:25 teamId 25 name "Arizona Cardinals" alias "ARI" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/ari.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "WEST"  
  
HSET team:26 teamId 26 name "Los Angeles Rams" alias "LA" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/lar.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "WEST"  
  
HSET team:27 teamId 27 name "San Francisco 49ers" alias "SF" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/sf.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "WEST"  
  
HSET team:28 teamId 28 name "Seattle Seahawks" alias "SEA" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/sea.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "WEST"  
```  
  
### NFC SOUTH (4 equipos)  
```redis  
HSET team:29 teamId 29 name "Atlanta Falcons" alias "ATL" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/atl.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "SOUTH"  
  
HSET team:30 teamId 30 name "Carolina Panthers" alias "CAR" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/car.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "SOUTH"  
  
HSET team:31 teamId 31 name "New Orleans Saints" alias "NO" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/no.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "SOUTH"  
  
HSET team:32 teamId 32 name "Tampa Bay Buccaneers" alias "TB" url "https://a.espncdn.com/combiner/i?img=/i/teamlogos/nfl/500/tb.png&scale=crop&cquality=40&location=origin&w=80&h=80" conference "NFC" division "SOUTH"  
```  
  
---  
  
## PARTE 2: CREAR ÍNDICES POR DIVISIÓN  
  
### AFC EAST  
```redis  
SADD teams:AFC:EAST 1 2 3 4  
```  
  
### AFC NORTH  
```redis  
SADD teams:AFC:NORTH 5 6 7 8  
```  
  
### AFC WEST  
```redis  
SADD teams:AFC:WEST 9 10 11 12  
```  
  
### AFC SOUTH  
```redis  
SADD teams:AFC:SOUTH 13 14 15 16  
```  
  
### NFC EAST  
```redis  
SADD teams:NFC:EAST 17 18 19 20  
```  
  
### NFC NORTH  
```redis  
SADD teams:NFC:NORTH 21 22 23 24  
```  
  
### NFC WEST  
```redis  
SADD teams:NFC:WEST 25 26 27 28  
```  
  
### NFC SOUTH  
```redis  
SADD teams:NFC:SOUTH 29 30 31 32  
```  
  
---  
  
## PARTE 3: CREAR LISTA GENERAL DE TODOS LOS EQUIPOS  
  
```redis  
SADD teams:all 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32  
```  
  
---  
  
## PARTE 4: COMANDOS PARA CONSULTAR  
  
### Obtener un equipo específico  
```redis  
HGETALL team:1  
# Resultado:  
# 1) "teamId"  
# 2) "1"  
# 3) "name"  
# 4) "Buffalo Bills"  
# 5) "alias"  
# 6) "BUF"  
# 7) "url"  
# 8) "https://..."  
# 9) "conference"  
# 10) "AFC"  
# 11) "division"  
# 12) "EAST"  
```  
  
### Obtener solo el nombre de un equipo  
```redis  
HGET team:1 name  
# Resultado: "Buffalo Bills"  
```  
  
### Obtener múltiples campos de un equipo  
```redis  
HMGET team:1 name alias conference division  
# Resultado:  
# 1) "Buffalo Bills"  
# 2) "BUF"  
# 3) "AFC"  
# 4) "EAST"  
```  
  
### Obtener IDs de todos los equipos de AFC EAST  
```redis  
SMEMBERS teams:AFC:EAST  
# Resultado: (1, 2, 3, 4)  
```  
  
### Obtener IDs de todos los equipos de NFC WEST  
```redis  
SMEMBERS teams:NFC:WEST  
# Resultado: (25, 26, 27, 28)  
```  
  
### Obtener TODOS los IDs de equipos  
```redis  
SMEMBERS teams:all  
# Resultado: (1, 2, 3, ..., 32)  
```  
  
### Contar equipos en una división  
```redis  
SCARD teams:AFC:EAST  
# Resultado: 4  
  
SCARD teams:NFC:WEST  
# Resultado: 4  
```  
  
### Verificar si un equipo está en una división  
```redis  
SISMEMBER teams:AFC:EAST 1  
# Resultado: 1 (true, Buffalo Bills está en AFC EAST)  
  
SISMEMBER teams:AFC:EAST 17  
# Resultado: 0 (false, Dallas Cowboys no está en AFC EAST)  
```  
  
---  
  
## PARTE 5: CONSULTAS AVANZADAS (Pipelining)  
  
### Obtener detalles de todos los equipos de AFC EAST  
```redis  
# Primero obtener los IDs  
SMEMBERS teams:AFC:EAST  
  
# Luego obtener cada equipo:  
HGETALL team:1  
HGETALL team:2  
HGETALL team:3  
HGETALL team:4  
```  
  
**En Python (automático):**  
```python  
import redis  
  
r = redis.Redis(host='localhost', port=6379, db=0)  
  
# Obtener IDs de equipos en AFC EAST  
team_ids = r.smembers('teams:AFC:EAST')  
  
# Obtener detalles de cada equipo  
teams = []  
for team_id in team_ids:  
    team = r.hgetall(f'team:{team_id}')    teams.append(team)```  
  
---  
  
## PARTE 6: ACTUALIZAR UN EQUIPO  
  
### Actualizar el nombre de un equipo  
```redis  
HSET team:1 name "Buffalo Bills Updated"  
```  
  
### Actualizar múltiples campos  
```redis  
HSET team:1 name "Buffalo Bills" alias "BUF" conference "AFC"  
```  
  
### Incrementar un campo numérico (si agregas puntos)  
```redis  
HINCRBY team:1 points 10  
```  
  
---  
  
## PARTE 7: ELIMINAR DATOS  
  
### Eliminar un equipo completo  
```redis  
DEL team:1  
```  
  
### Eliminar un campo específico de un equipo  
```redis  
HDEL team:1 url  
```  
  
### Eliminar todos los equipos  
```redis  
DEL team:1 team:2 team:3 ... team:32  
# O mejor:  
KEYS team:* | xargs redis-cli DEL  
```  
  
### Limpiar todo (equipos + índices)  
```redis  
FLUSHDB  
# Elimina toda la base de datos actual  
```  
  
---  
  
## PARTE 8: LISTAR TODAS LAS CLAVES (para verificar)  
  
```redis  
KEYS *  
# Muestra todas las claves: team:1, team:2, ..., teams:AFC:EAST, teams:all, etc.  
  
KEYS team:*  
# Muestra solo los equipos: team:1, team:2, ..., team:32  
  
KEYS teams:*  
# Muestra solo los índices: teams:AFC:EAST, teams:AFC:NORTH, etc.  
```  
  
---  
  
## RESUMEN DE ESTRUCTURA EN REDIS  
  
```  
Equipos (Hashes):  
├── team:1 → {teamId, name, alias, url, conference, division}  
├── team:2 → {teamId, name, alias, url, conference, division}  
├── ...  
└── team:32 → {teamId, name, alias, url, conference, division}  
  
Índices por División (Sets):  
├── teams:AFC:EAST → {1, 2, 3, 4}  
├── teams:AFC:NORTH → {5, 6, 7, 8}  
├── teams:AFC:WEST → {9, 10, 11, 12}  
├── teams:AFC:SOUTH → {13, 14, 15, 16}  
├── teams:NFC:EAST → {17, 18, 19, 20}  
├── teams:NFC:NORTH → {21, 22, 23, 24}  
├── teams:NFC:WEST → {25, 26, 27, 28}  
├── teams:NFC:SOUTH → {29, 30, 31, 32}  
└── teams:all → {1, 2, ..., 32}  
```