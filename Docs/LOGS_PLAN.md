# Logs Plan - Health Robot

_Stratégie de logging pour observabilité du robot autonome: traces pour valider livraisons, déboguer collisions, monitorer batterie, et auditeer actions personnel._

**Contexte**: Robot ROSMASTER M3 Pro + ROS2 + MQTT + FastAPI + React  
**Objectif**: Tracer toutes les événements UC-01 à UC-08 pour debug/audit

---

## Principes

1. **Traçabilité**: Chaque action importante doit être loggée
2. **Observabilité**: Logs suffisants pour diagnostiquer problèmes en prod
3. **Performance**: Logging asynchrone pour ne pas bloquer les opérations
4. **Sécurité**: Pas de données sensibles (mots de passe, tokens)
5. **Centralisation**: Tous les logs au même endroit pour analyser

---

## Architecture logging

```
┌──────────────────────┐
│  Frontend (Console)  │
└──────────┬───────────┘
           │
           │ HTTP POST /api/logs
           ↓
┌──────────────────────┐          ┌──────────────────┐
│  Backend (FastAPI)   │────────→ │  Log Collector   │
└──────────┬───────────┘          └────────┬─────────┘
           │                                │
           │ MQTT Publish                   │
           ↓                                │
┌──────────────────────┐                   │
│  Robot Hardware      │                   │
└──────────┬───────────┘                   │
           │                                │
           │ MQTT Publish                   │
           └────────────────────────────────┘
                      │
                      ↓
            ┌──────────────────┐
            │ Storage centralisé│
            │  (File/DB/Stack) │
            └────────┬─────────┘
                     │
                     ↓
            ┌──────────────────┐
            │  Monitoring &    │
            │  Analytics       │
            └──────────────────┘
```

---

## Signaux & Événements par composant

### 1. Frontend Logs (React/Browser)

#### Catégories

| Catégorie | Sévérité | Exemple | Use Case |
|-----------|----------|---------|----------|
| **Commande livraison** | INFO | Mission created (room 302, matériel) | Audit UC-01/UC-02 |
| **Confirmation livraison** | INFO | User confirmed delivery (room 302) | Audit UC-01/UC-02 |
| **API Calls** | INFO/ERROR | POST /api/mission 200, GET /api/robot/status 500 | Tracer requêtes |
| **MQTT Connection** | INFO/WARN | WebSocket connected, Disconnect timeout | Connectivité |
| **Alerte urgence** | ERROR | EMERGENCY STOP activated | UC-08 |
| **Alerte batterie** | WARN | Battery critical (15%), notification sent | UC-07 |
| **Alerte collision** | WARN | Collision detected (type: person), Robot stopped | UC-04 |
| **Navigation réelle-temps** | DEBUG | Robot position update: x=1.5, y=2.3 | UC-03 |
| **Errors** | ERROR | Cannot fetch robot status (3 retries failed) | Debugger |
| **Performance** | WARN | API call took 2.5s (expected < 1s) | Optimiser |
| **User journey** | INFO | User navigated from dashboard → delivery_form → confirmation | Analytics |
| **Device info** | INFO | Browser: Chrome, Device: iPad Pro, OS: iPadOS | Compat |

**Utilisateurs ciblés**: Marie (soignant), Paul (cuisine) → logs simples à lire sur tablette

#### Format log (JSON structuré)
```javascript
{
  timestamp: "2026-02-25T10:30:45.123Z",
  level: "INFO|WARN|ERROR",
  component: "DeliveryForm|RobotTracker|BatteryAlert",
  action: "mission.create|mission.confirm|battery.alert_critical",
  request_id: "req-uuid",  // Traçabilité chaîne
  user_id: "user-uuid",
  user_role: "soignant|cuisine|infirmier",
  details: {
    mission_id: "123",
    room: "302",
    type: "materiel|repas",
    battery_level: 15,
    duration_ms: 1250,
    error_message: null
  },
  session: "user-session-uuid"
}
```

#### Implémentation
- **Logger Utility**: `src/utils/logger.ts` (custom wrapper)
- **Destination**: POST `/api/logs` → Backend (batch toutes les 10s)
- **WebSocket**: Souscription directe aux topics:
  - `/robot/status` → Position temps-réel
  - `/robot/battery` → Alerte batterie
  - `/delivery/{id}/update` → État livraison
  - `/robot/collision` → Alerte obstacle
  - `/robot/emergency` → Arrêt urgence
- **Fréquence**: Immédiat pour ERROR/CRITICAL, batch pour INFO

---

### 2. Backend Logs (FastAPI/Python)

#### Catégories

| Catégorie | Sévérité | Exemple | Use Case |
|-----------|----------|---------|----------|
| **Commande reçue** | INFO | POST /api/mission 200, mission_id=123 room=302 | UC-01/UC-02 |
| **Validation commande** | INFO/WARN | Room 302 invalid (not found), Room valid ✓ | UC-01/UC-02 validation |
| **MQTT publish** | INFO | Published /delivery/123/start to broker | UC-01/UC-02 ↔ robot |
| **MQTT subscribe** | INFO | Received /delivery/123/complete {success:true} | UC-01/UC-02 confirmation |
| **Robot status** | INFO | Robot battery=45%, position=<1.5,2.3>, state=navigating | UC-03 suivi |
| **Collision détectée** | ERROR | Collision signal from MQTT /robot/collision {type:person} | UC-04 sécurité |
| **Batterie critique** | WARN | Robot battery=18% < 20%, triggering return_to_base | UC-07 |
| **Arrêt d'urgence** | CRITICAL | EMERGENCY STOP received, motors disabled | UC-08 |
| **Timeout MQTT** | ERROR | No response from robot for 30s, mission timeout | Détecter robot offline |
| **Database** | WARN | Query duration=250ms > 100ms threshold | Optimiser DB |
| **API latency** | INFO | Response time 1.2s > 1s target | Performance monitoring |
| **Auth** | WARN | Failed auth attempt, user not found | Sécurité |
| **Error handling** | ERROR | Exception in routes/delivery.py line 42 | Debug |

**Utilisateurs ciblés**: Dev team, DevOps, Infirmier → logs détaillés pour debug

#### Format log (JSON structuré avec traçabilité)
```python
{
    "timestamp": "2026-02-25T10:30:45.123Z",
    "level": "INFO|WARN|ERROR|CRITICAL",
    "service": "delivery_api|mqtt_handler|robot_status|safety",
    "action": "mission.create|mission.confirm|robot.status.update|collision.detected|battery.critical",
    "request_id": "req-abc123",  # Traçabilité chaîne complète
    "user_id": "user-uuid",
    "user_role": "soignant|cuisine|infirmier",
    "duration_ms": 145,
    "http_status": 200,
    "mqtt_topic": "/delivery/123/start",  # Si MQTT
    "details": {
        "mission_id": "123",
        "room": "302",
        "cargo_type": "materiel|repas",
        "robot_id": "ROSMASTER-01",
        "robot_state": "navigating|idle|charging",
        "battery_level": 45,
        "position": {"x": 1.5, "y": 2.3},
        "collision_type": "person|obstacle",  # Si collision
        "error": null  # Stack trace si ERROR
    }
}
```

#### Implémentation
```python
# app/core/logging_config.py
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "level": record.levelname,
            "service": record.name,
            "action": getattr(record, 'action', ''),
            "message": record.getMessage(),
            "request_id": getattr(record, 'request_id', ''),
        }
        return json.dumps(log_data)

logger = logging.getLogger(__name__)
handler = logging.FileHandler('logs/backend.log')
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

# Dans les routes
logger.info("mission created", extra={
    "action": "mission.create",
    "request_id": request_id,
    "mission_id": mission.id,
    "room": room
})
```

#### Endpoints pour logs frontend→backend
- `POST /api/logs` - Backend reçoit et stocke logs du frontend
- `GET /api/logs` - Récupérer historique complet
- `GET /api/logs/missions` - Historique livraisons (filtré par utilisateur)
- `GET /api/logs/errors` - Seulement erreurs + warnings
- `GET /api/logs/filter?level=ERROR&action=mission.create` - Filtrer par critères

---

### 3. Robot Hardware Logs (ROS2 Nodes sur Jetson Nano/Orin NX)

#### Signaux par Node ROS2

| ROS2 Node | Signal | Sévérité | Exemple | Use Case |
|-----------|--------|----------|---------|----------|
| **mqtt_node** | Connection | INFO/ERROR | Connected to broker, Connection lost | Connectivité |
| - | Message received | DEBUG | Received /delivery/123/start | UC-01/UC-02 |
| - | Message published | DEBUG | Published /robot/status | UC-03 suivi |
| **navigation_node** (Nav2) | Goal received | INFO | Navigation goal: room_302 | UC-01/UC-02 |
| - | Path planned | DEBUG | Planned path (12 waypoints) | UC-03 |
| - | Navigation started | INFO | Starting autonome navigation | UC-03 |
| - | Goal reached | INFO | Reached destination (tolerance 0.3m) | UC-01/UC-02 |
| - | Navigation failed | ERROR | Path blocked, no solution found | UC-03 erreur |
| **obstacle_avoidance_node** | Obstacle detected | WARN | Obstacle detected distance=0.8m | UC-04 |
| - | Person detected | CRITICAL | Person too close! EMERGENCY STOP | UC-04 sécurité |
| - | Avoidance strategy | DEBUG | Attempting left bypass | UC-04 |
| - | Avoidance failed | ERROR | Cannot bypass obstacle, requesting help | UC-04 erreur |
| **battery_monitor_node** | Battery status | INFO | Battery: 75%, Discharging at 2A | UC-07 |
| - | Battery low | WARN | Battery: 22% < 25% threshold | UC-07 alerte |
| - | Battery critical | CRITICAL | Battery: 15% < 20%, returning to base | UC-07 |
| - | Charging started | INFO | Charging started at dock | UC-05 |
| - | Charging complete | INFO | Battery 100%, fully charged | UC-05 |
| **safety_node** | Emergency stop | CRITICAL | EMERGENCY STOP ACTIVATED (button) | UC-08 |
| - | Sensor failure | ERROR | Lidar communication lost | Sécurité |
| - | Motor status | INFO | Motors online and healthy | Health check |
| **vision_node** | Detection | DEBUG | Detected person at x=1.2, confidence=0.95 | UC-04 vision |
| - | Classification error | WARN | Cannot classify obstacle type | UC-04 |
| **slam_node** | Mapping | DEBUG | Map updated (2150 features) | Cartographie |
| - | Localization | INFO | Localization confidence: 0.98 | Navigation health |
| - | Loop closure | DEBUG | Loop closure detected (keyframe 45) | SLAM quality |
| **delivery_node** | Task received | INFO | Delivery task: mission_123 origin=kitchen destination=room_302 | UC-01/UC-02 |
| - | Task started | INFO | Starting delivery mission | UC-01/UC-02 |
| - | Cargo detected | INFO | Cargo loaded, ready to navigate | UC-02 repas |
| - | Cargo status | DEBUG | Cargo weight: 2.5kg < max 5kg | UC-01/UC-02 |
| - | Delivery complete | INFO | Cargo delivered successfully | UC-01/UC-02 |
| - | Task canceled | WARN | Delivery canceled by user | Cancellation |

**Utilisateurs ciblés**: Robot dev team, Roboticists → logs de debug détaillés

#### Format MQTT Topics (Robot → Backend)

**Tous les topics publiés par le robot (via mqtt_node bridge ROS2→MQTT)**:

```json
Topic: /robot/status
Payload: {
  "timestamp": "2026-02-25T10:30:45.123Z",
  "robot_id": "ROSMASTER-01",
  "state": "navigating|idle|charging|emergency_stop",
  "position": {"x": 1.5, "y": 2.3, "orientation": 45},
  "velocity": {"linear": 0.3, "angular": 0.15},
  "current_mission": "mission-123",
  "confidence": 0.98
}

Topic: /robot/battery
Payload: {
  "timestamp": "2026-02-25T10:30:45.123Z",
  "level": 45,
  "voltage": 11.5,
  "current_draw": 2.1,
  "status": "discharging|charging",
  "eta_charge_minutes": null,
  "health": "good|degraded|critical"
}

Topic: /robot/collision
Payload: {
  "timestamp": "2026-02-25T10:30:45.123Z",
  "detected": true,
  "type": "person|obstacle|wall",
  "distance": 0.8,
  "location": {"x": 1.5, "y": 2.3},
  "action_taken": "stopped|bypassing|waiting"
}

Topic: /delivery/{mission_id}/status
Payload: {
  "timestamp": "2026-02-25T10:30:45.123Z",
  "mission_id": "123",
  "state": "received|navigating|arrived|cargo_delivery|completed|failed",
  "current_room": "302",
  "elapsed_seconds": 145,
  "cargo_weight": 2.5
}

Topic: /robot/logs (Debug stream)
Payload: {
  "timestamp": "2026-02-25T10:30:45.123Z",
  "level": "DEBUG|INFO|WARN|ERROR",
  "node": "navigation_node|obstacle_avoidance|delivery_node",
  "message": "Navigation goal accepted: coordinates [1.5, 2.3]",
  "details": {}
}
```

**QoS Settings**:
- Status topics: QoS 1 (at least once)
- Critical alerts (collision, emergency): QoS 2 (exactly once)
- Debug: QoS 0 (fire and forget)

---

## Stockage & Persistence

### Option 1: File-based (Simple, pour MVP)

```bash
# Structure
logs/
├── 2026-02-25/
│   ├── frontend.log
│   ├── backend.log
│   └── robot.log
└── 2026-02-26/
    ├── ...
```

**Avantages**: Simple, pas de dépendance, rotation facile  
**Inconvénients**: Pas de search facile

### Option 2: Database (Production)

```sql
CREATE TABLE logs (
    id UUID PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    level VARCHAR(10),
    service VARCHAR(50),
    action VARCHAR(100),
    request_id UUID,
    details JSONB,
    user_id UUID,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX (timestamp, level, service)
);
```

**Avantages**: Searchable, queryable, retentions policies  
**Inconvénients**: Dépendance DB

### Option 3: Stack ELK (Production scalable)

```
Application → Logstash → Elasticsearch → Kibana
                ↓             ↓
         Format & Filter   Store & Index
                              ↓
                        OpenSearch/ELK
```

**Avantages**: Scalable, powerful analytics  
**Inconvénients**: Complexe, ressources

### Recommandation MVP
**Commencer par Option 1 (File-based)**, migrer à Option 2 (DB) pour prod.

---

## Retention & Rotation

| Logs | Rétention | Rotation |
|------|-----------|----------|
| Frontend | 30 jours | Daily |
| Backend | 90 jours | Daily |
| Robot Hardware | 365 jours | Weekly |

```bash
# Rotation logs (logrotate ou custom script)
/app/logs/backend.log {
    daily
    rotate 90
    compress
    delaycompress
    missingok
    notifempty
}
```

---

## Requestid Tracing (Chaîne complète)

**Objectif**: Suivre une mission de bout en bout pour debug.

```
Frontend génère request_id unique
  ↓
Frontend: POST /api/mission {request_id: "req-abc123", room: "302", ...}
  ↓
Backend reçoit, log: action=mission.create request_id=req-abc123
  ↓
Backend publish MQTT: /delivery/mission-123/start {request_id: "req-abc123", ...}
  ↓
Robot reçoit via mqtt_node, log: request_id=req-abc123 event=mission_received
  ↓
Robot navigation_node log: request_id=req-abc123 navigation_started
  ↓
Robot navigation_node log: request_id=req-abc123 goal_reached
  ↓
Robot publish: /delivery/mission-123/complete {request_id: "req-abc123", success: true}
  ↓
Backend reçoit, log: request_id=req-abc123 action=mission.confirm
  ↓
Frontend POST /api/mission/123/confirm {request_id: "req-abc123"}
  ↓
Backend log: request_id=req-abc123 action=mission.completed
```

**Traçabilité**: Chercher `request_id=req-abc123` dans tous les logs → trace complète

---

## Observabilité des problèmes courants

### Problème: Robot ne répond pas / Offline

**Symptôme**: Bouton "Livrer" déclenche l'error frontend "Robot not responding"

**Checklist logs**:
```
1. Backend logs:
   GET /api/logs?action=robot.status&level=ERROR (dernier 5 min)
   Chercher: "MQTT timeout", "No connection to broker"

2. MQTT broker logs:
   docker logs health-robot_mosquitto
   Chercher: "Client disconnected", connection errors

3. Robot logs (sur Jetson):
   rostopic echo /robot/status (should pub every 1 Hz)
   Chercher: Rien? mqtt_node pas lancé?

4. Trace complète:
   GET /api/logs?request_id=<mission-id>&level=ERROR
   Doit montrer:
   - mission.create OK ✓
   - MQTT publish /delivery/XYZ/start OK ✓
   - [MISSING] MQTT subscribe /delivery/XYZ/complete ✗
```

**Actions**:
- Vérifier Mosquitto run: `docker ps | grep mosquitto`
- Vérifier robot connecté: `mosquitto_sub -h localhost -t /robot/status`
- Redémarrer mqtt_node sur robot

---

### Problème: Mission échoue à 50% (robot s'arrête en route)

**Symptôme**: Frontend affiche "Navigation in progress" puis timeout → error

**Checklist logs**:
```
1. Récupérer request_id de la mission échouée:
   GET /api/logs?action=mission.create (trouver "room 302")
   Ex: request_id = req-xyz789

2. Tracer la chaîne complète:
   GET /api/logs?request_id=req-xyz789
   Doit montrer:
   - Backend: mission.create ✓
   - Backend: MQTT publish /delivery/123/start ✓
   - Robot: mission_received ✓
   - Robot: navigation_started ✓
   - Robot: [MISSING] goal_reached ✗ STOPPED HERE

3. Chercher pourquoi navigation s'arrête:
   Robot logs (on Jetson):
   rostopic echo /diagnostics
   Chercher: "stuck", "navigation failed", "cancel"
   
   Ou MQTT topics:
   mosquitto_sub -t /robot/collision (collision détectée?)
   mosquitto_sub -t /robot/battery (batterie critique?)
```

**Causes possibles et logs**:
- **Obstacle bloque**: `/robot/collision` avec `type: person|obstacle`
- **Batterie faible**: `/robot/battery` < 20% + action: returning_to_base
- **Nav2 timeout**: `navigation_node` log: "Goal unreachable"
- **Perte MQTT**: mqtt_node logs: "reconnecting"
- **User cancellation**: Frontend log: `mission.cancel`

---

### Problème: Collision immédiate (robot écrase personne)

**Symptôme**: Incident sécurité! Vérifier urgence logs

**Checklist logs (CRITICAL)**:
```
1. IMMÉDIAT: Chercher emergency stop logs:
   GET /api/logs?level=CRITICAL&action=emergency_stop
   Doit montrer: timestamp, qui a appuyé, pourquoi

2. Collision logs:
   GET /api/logs?action=collision.detected
   Doit montrer: distance, type (person), position, action_taken

3. Robot safety logs:
   Robot logs: safety_node, obstacle_avoidance_node
   Chercher: "Person detected at distance 0.5m"
   ID: "EMERGENCY STOP TRIGGERED"

4. Timeline:
   Quand? Où (position x,y)? Qui faisait quoi (room)?
   Logs frontend: utilisateur était sur quelle page?

5. Root cause analysis:
   - Lidar calibration OK? (rostopic echo /scan)
   - À quelle vitesse? (v_linear > 0.5 m/s = écart specs)
   - Personne s'est elle approchée? (vs robot qui a foncé)
```

**Actions**:
- Arrêter tous les robots: POST /api/emergency
- Inspection physique du robot + Lidar
- Review obstacle_avoidance_node code

---

### Problème: Batterie n'envoie pas alerte

**Symptôme**: Robot s'arrête car batterie vide, pas de notification frontend

**Checklist logs**:
```
1. Robot battery_monitor node:
   rostopic echo /robot/battery | grep "level"
   Logs: battery_monitor_node should pub every 500ms

2. MQTT topic:
   mosquitto_sub -t /robot/battery (arrive au broker?)

3. Backend MQTT subscription:
   GET /api/logs?action=battery.critical
   Doit voir: "Battery 15% < 20% threshold - returning_to_base"

4. Frontend WebSocket:
   Browser console: check WebSocket connected? Check /robot/battery subscription?

5. Timing:
   Quand alerte a été sentie? (timestamp)
   Robot a-t-il eu le temps de retourner? (ETA en logs)
```

---

### Problème: Lenteur API (response > 2s)

**Symptôme**: Interface figée, "Livrer" button slow

**Checklist logs**:
```
1. Backend perf logs:
   GET /api/logs?level=WARN&service=delivery_api&duration_ms>2000
   Chercher: POST /api/mission slow endpoints

2. Bottleneck?
   - MQTT publish slow? (topic /delivery/XYZ/start timeout?)
   - Database query slow? (si persistence activée)
   - Validation slow? (room existence check)
   
3. Trace spécifique:
   GET /api/logs?request_id=<slow-mission-id>
   Timing breakdown:
   - mission.validation_start → validation_end: X ms
   - MQTT_publish_start → MQTT_publish_end: X ms
   - Total: X ms

4. Solution:
   Si MQTT slow: check Mosquitto broker (mosquitto_sub lag)
   Si DB slow: optimize query
   Si validation slow: cache room list
```

---

## Alerting & Monitoring (Seuils & Actions)

### Seuils d'alerte

| Condition | Sévérité | Action | Destinataire |
|-----------|----------|--------|---------------|
| 5+ ERROR en 5 min | RED | Log file + admin dashboard | DevOps team |
| Aucun /robot/status depuis 30s | RED | Robot offline → disable UI deploy button | Frontend users + Infirmier |
| Battery < 15% | ORANGE | Notification + UI alerte + return_to_base = AUTO | Frontend users |
| Battery < 5% | RED | Emergency warning + immediate dock | Frontend users |
| Collision detected | RED | STOP robot + log incident + notification | Infirmier + DevOps |
| API response > 2s | YELLOW | Log slowness + investigate | DevOps team |
| MQTT dropped (1 min) | RED | Broker down alert | DevOps team |
| Database connection fail | CRITICAL | Immediate alert + fallback to in-memory logs | DevOps team |
| 3+ failed missions in 1h | YELLOW | Investigate why (obstacles? nav issues?) | Infirmier |

### Actions par niveau

```
CRITICAL:
  → Slack/Email/SMS to on-call engineer
  → Disable robot until fixed
  → Page infirmier:

RED/ERROR:
  → Slack alert
  → Visible in admin dashboard
  → Log to analytics

ORANGE/WARN:
  → Log to file
  → Visible in stats/reports

GREEN/INFO:
  → Log to file only (high volume)
```

---

## Setup logging par service

### Backend (FastAPI) — Structured JSON logging
```python
# app/core/logging_config.py
import logging
import json
from datetime import datetime
from logging.handlers import RotatingFileHandler

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "level": record.levelname,
            "service": record.name.split('.')[0],  # app.routes → app
            "action": getattr(record, 'action', ''),
            "message": record.getMessage(),
            "request_id": getattr(record, 'request_id', ''),
            "user_id": getattr(record, 'user_id', ''),
            "duration_ms": getattr(record, 'duration_ms', None),
            "details": getattr(record, 'details', {}),
        }
        # Remove None values
        return json.dumps({k: v for k, v in log_data.items() if v})

def setup_logging():
    logger = logging.getLogger()
    logger.setLevel(logging.DEBUG)
    
    # File handler with rotation (daily, keep 30 days)
    handler = RotatingFileHandler(
        'logs/backend.log',
        maxBytes=10485760,  # 10MB
        backupCount=30
    )
    handler.setFormatter(JSONFormatter())
    logger.addHandler(handler)
    
    # Also console for development
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(JSONFormatter())
    logger.addHandler(console_handler)

# app/main.py
from app.core.logging_config import setup_logging
setup_logging()

# In routes - example
import logging
logger = logging.getLogger(__name__)

@app.post("/api/mission")
async def create_mission(req: MissionRequest, bg_tasks: BackgroundTasks):
    request_id = str(uuid.uuid4())
    start_time = time.time()
    
    try:
        # Validation
        logger.info("validating mission", extra={
            "action": "mission.validate",
            "request_id": request_id,
            "room": req.room,
            "user_id": current_user.id
        })
        
        # Create mission
        mission = Mission(room=req.room, user_id=current_user.id)
        db.add(mission)
        db.commit()
        
        # Publish MQTT
        await mqtt_client.publish(
            f"/delivery/{mission.id}/start",
            json.dumps({"request_id": request_id, ...})
        )
        
        duration_ms = int((time.time() - start_time) * 1000)
        logger.info("mission created", extra={
            "action": "mission.create",
            "request_id": request_id,
            "mission_id": mission.id,
            "duration_ms": duration_ms,
            "user_id": current_user.id
        })
        return {"mission_id": mission.id, "request_id": request_id}
        
    except Exception as e:
        logger.error("mission creation failed", extra={
            "action": "mission.create_failed",
            "request_id": request_id,
            "error": str(e),
            "user_id": current_user.id,
            "duration_ms": int((time.time() - start_time) * 1000)
        })
        raise HTTPException(status_code=500, detail=str(e))
```

### Frontend (React/TypeScript) — Client-side logging
```typescript
// src/utils/logger.ts
import { useCallback } from 'react'

interface LogEntry {
  timestamp: string
  level: 'DEBUG' | 'INFO' | 'WARN' | 'ERROR'
  component: string
  action: string
  request_id?: string
  user_id?: string
  details?: Record<string, any>
  session: string
}

const LOG_BUFFER: LogEntry[] = []
const BATCH_SIZE = 10
const BATCH_INTERVAL_MS = 10000

let batchTimer: NodeJS.Timeout | null = null

function flushLogs() {
  if (LOG_BUFFER.length === 0) return
  
  const batch = LOG_BUFFER.splice(0, BATCH_SIZE)
  fetch('/api/logs', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(batch)
  }).catch(err => console.error('Log flush failed:', err))
}

export function useLogger() {
  return useCallback((entry: LogEntry) => {
    // Local console always
    console.log(`[${entry.level}] ${entry.component}:`, entry)
    
    // Buffer for batching
    LOG_BUFFER.push(entry)
    
    // Flush immediately if ERROR/CRITICAL
    if (['ERROR', 'CRITICAL'].includes(entry.level)) {
      flushLogs()
    }
    // Flush when buffer full
    else if (LOG_BUFFER.length >= BATCH_SIZE) {
      flushLogs()
    }
    // Schedule flush if not already scheduled
    else if (!batchTimer) {
      batchTimer = setTimeout(() => {
        flushLogs()
        batchTimer = null
      }, BATCH_INTERVAL_MS)
    }
  }, [])
}

// Usage in component
import { useLogger } from '@/utils/logger'

export function DeliveryForm() {
  const logger = useLogger()
  const sessionId = useSession().id
  
  const handleSubmit = async (room: string, type: string) => {
    const requestId = crypto.randomUUID()
    
    logger({
      timestamp: new Date().toISOString(),
      level: 'INFO',
      component: 'DeliveryForm',
      action: 'mission.create_start',
      request_id: requestId,
      details: { room, type },
      session: sessionId
    })
    
    try {
      const res = await fetch('/api/mission', {
        method: 'POST',
        body: JSON.stringify({ room, type, request_id: requestId })
      })
      
      if (res.ok) {
        const data = await res.json()
        logger({
          timestamp: new Date().toISOString(),
          level: 'INFO',
          component: 'DeliveryForm',
          action: 'mission.create_success',
          request_id: requestId,
          details: { mission_id: data.mission_id, room, type },
          session: sessionId
        })
      } else {
        throw new Error(`API failed: ${res.status}`)
      }
    } catch (err) {
      logger({
        timestamp: new Date().toISOString(),
        level: 'ERROR',
        component: 'DeliveryForm',
        action: 'mission.create_failed',
        request_id: requestId,
        details: { error: String(err), room, type },
        session: sessionId
      })
    }
  }
  
  return (...)
}

// MQTT WebSocket subscriptions
import { useMQTT } from '@/hooks/useMQTT'

export function RobotTracker() {
  const logger = useLogger()
  const sessionId = useSession().id
  const mqtt = useMQTT()
  
  useEffect(() => {
    // Subscribe to robot updates
    mqtt.subscribe('/robot/status', (payload) => {
      const status = JSON.parse(payload)
      logger({
        timestamp: new Date().toISOString(),
        level: 'DEBUG',
        component: 'RobotTracker',
        action: 'robot.status_update',
        details: { state: status.state, battery: status.battery },
        session: sessionId
      })
      setRobotStatus(status)
    })
    
    mqtt.subscribe('/robot/battery', (payload) => {
      const battery = JSON.parse(payload)
      if (battery.level < 20) {
        logger({
          timestamp: new Date().toISOString(),
          level: 'WARN',
          component: 'RobotTracker',
          action: 'battery.low_alert',
          details: { level: battery.level },
          session: sessionId
        })
      }
    })
  }, [])
  
  return (...)
}
```

---

## Alerting & Monitoring

### Seuils d'alerte

| Condition | Sévérité |
|-----------|----------|
| 5+ erreurs en 5 min | WARN |
| Aucun heartbeat robot depuis 30s | ERROR |
| Battery < 15% | WARN |
| API response > 2s | WARN |
| Database connection failed | CRITICAL |

### Actions
```bash
ERROR → Slack/Email
WARN → Dashboard update + Log
INFO → Store seulement
```

---

## Checklist implémentation (par Phase)

### Phase 1 (23/02 → 15/03) — Basic logging
- [ ] Logger wrapper Frontend `src/utils/logger.ts`
- [ ] Endpoint Backend `POST /api/logs` pour recevoir logs frontend
- [ ] Format JSON consistent (avec request_id, user_id, action)
- [ ] File-based logs with rotation (`logs/backend.log`, `logs/frontend.log`)
- [ ] MQTT topics pour robot status/battery/collision (définis)
- [ ] ROS2 mqtt_node bridge (publish vers backend)
- [ ] Safety: NO passwords/tokens in logs
- [ ] Test logs avec une mission complète (end-to-end)

### Phase 2 (16/03 → 26/04)
- [ ] Structured JSON logging validé en production
- [ ] Request ID tracing dans MQTT topics
- [ ] Robot ROS2 nodes logging (navigation, obstacle, battery)
- [ ] Logs volumes manageable (rotation working)
- [ ] Retention policy pour chaque log type

### Phase 3+ (27/04 →)
- [ ] Dashboard/CLI pour visualiser logs (grep + tail suffisant? ou Kibana?)
- [ ] Alerting seuils (5+ errors → slack)
- [ ] Analytics: Success rate missions, avg delivery time
- [ ] Migration to PostgreSQL (si volume trop grand)
- [ ] ELK/OpenSearch (if full enterprise needed)

---

## Ressources & Outils

- **Logging Library Frontend**: `pino-js` ou `winston`
- **Logging Library Backend**: Built-in `logging` Python
- **Storage**: File system → PostgreSQL → ELK Stack
- **Visualization**: Custom CLI ou Kibana/OpenSearch
- **Monitoring**: Prometheus + Grafana (future)

---

## Résumé hiérarchie logs

```
┌─ Frontend (React) ─────────────────────────────┐
│ - Mission creation/confirmation (USER actions) │
│ - API call tracing (HTTP status, latency)       │
│ - MQTT WebSocket connection (real-time updates) │
│ - Battery/Collision alerts (visual + notify)    │
│ Flow: Browser → fetch('/api/logs') → Backend   │
└────────────────────────────────────────────────┘
                        │
                        ↓ (aggregated)
┌─ Backend (FastAPI) ──────────────────────────────┐
│ - Mission orchestration (validation, MQTT pub)   │
│ - MQTT subscription (status, battery, collision) │
│ - API timings (slow query detection)             │
│ - Error handling (validation, DB, MQTT errors)   │
│ Flow: Backend → logs/backend.log (rotating)      │
└──────────────────────────────────────────────────┘
                        │
                        ↓ (MQTT bridge)
┌─ Robot ROS2 ─────────────────────────────────────┐
│ - 8 Nodes publishing detailed status/debug info   │
│ - Real-time sensor data (lidar, battery, IMU)    │
│ - Navigation (goal received, path planned, etc)   │
│ - Safety (collision detection, emergency stop)   │
│ - Delivery mission state (received→complete)     │
│ Flow: ROS2 nodes → mqtt_node → MQTT broker       │
└──────────────────────────────────────────────────┘
```

**Chaîne de responsabilité**:
- Frontend: Logs utilisateur (Marie/Paul actions)
- Backend: Logs système + orchestration
- Robot: Logs détaillés technical (debug team)
- Infirmier: Accès à tous (dashboard)

---

**Dernière mise à jour**: 25 février 2026  
**Basé sur**: CDC-technique.md (Use Cases UC-01 à UC-08)  
**Propriétaire**: Team DevOps/Robotics  
**À déléguer à**: [Personne responsable logging & monitoring]  
**Pré-requis**: Après Phase 1 (ROS2 setup)
