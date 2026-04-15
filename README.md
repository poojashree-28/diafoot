# DiaFoot 2.0 — PHC Digital Platform

A full-stack web application for diabetic foot monitoring integrated with the DiaFoot hardware prototype and the PHC Digital Health Platform.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, Vanilla JavaScript, Chart.js |
| Backend | Python 3.x + Flask |
| Database | SQLite (drop-in MySQL-compatible schema) |
| API Integration | REST JSON endpoints |

---

## Folder Structure

```
diafoot2/
├── run.py                        # Entry point
├── requirements.txt
├── backend/
│   ├── __init__.py               # App factory
│   ├── models/
│   │   └── db.py                 # SQLite connection + seed data
│   └── routes/
│       ├── auth.py               # Doctor login / session
│       ├── dashboard.py          # PHC Dashboard APIs
│       ├── patients.py           # Patient CRUD
│       ├── sensors.py            # Prototype integration: POST /sensor-data
│       ├── attendance.py         # Check-in / check-out / absenteeism
│       ├── ddhs.py               # Central monitoring
│       └── alerts.py             # Alert management
├── frontend/
│   ├── templates/
│   │   └── index.html            # Single-page app shell
│   └── static/
│       ├── css/main.css          # Complete design system
│       └── js/
│           ├── api.js            # API helper
│           ├── app.js            # Router, auth, navigation
│           └── pages/
│               ├── phc-dashboard.js
│               ├── attendance.js
│               ├── ddhs-dashboard.js
│               ├── patients.js
│               └── alerts.js
└── database/
    └── schema.sql                # Database schema + seed data
```

---

## Setup & Run

### 1. Install Dependencies
```bash
pip install flask
```

### 2. Run the Server
```bash
python run.py
```
App starts at: **http://localhost:5000**

### 3. Demo Login Accounts

| Role | Employee ID | Password |
|---|---|---|
| DDHS Admin | DDHS001 | admin123 |
| PHC Admin | PHC001 | admin123 |
| Doctor | DOC001 | admin123 |

---

## API Reference

### Auth
```
POST /api/auth/login       { employee_id, password }
POST /api/auth/logout
GET  /api/auth/me
```

### Prototype Integration (Hardware → Server)
```
POST /api/sensor-data
{
  "patient_id":     "PAT001",
  "temperature":    36.5,
  "pressure_left":  45.2,
  "pressure_right": 43.8,
  "moisture":       55.0,
  "wound_status":   "none"
}
→ Stores data, runs threshold checks, generates alerts automatically
```

### PHC Dashboard
```
GET /api/dashboard/phc/<phc_id>
GET /api/dashboard/daily-report/<phc_id>
```

### Patients
```
GET  /api/patients/?phc_id=&search=&risk=
GET  /api/patients/<patient_id>
POST /api/patients/
```

### Alerts
```
GET  /api/alerts/?phc_id=&severity=&acknowledged=0
POST /api/alerts/<id>/acknowledge
```

### Attendance
```
POST /api/attendance/checkin          { doctor_id }
POST /api/attendance/checkout         { doctor_id }
GET  /api/attendance/status/<doctor_id>
GET  /api/attendance/history/<doctor_id>
GET  /api/attendance/phc/<phc_id>/today
POST /api/attendance/generate-absenteeism-alerts
GET  /api/attendance/absenteeism-alerts?phc_id=
```

### DDHS Central
```
GET /api/ddhs/overview
GET /api/ddhs/phc-analytics
GET /api/ddhs/attendance-chart
GET /api/ddhs/alerts-trend
GET /api/ddhs/absenteeism-report
GET /api/ddhs/critical-cases
```

---

## Alert Threshold Logic

| Parameter | Warning | Critical |
|---|---|---|
| Temperature | 30–35°C | <28°C or >38°C |
| Pressure Left | 10–80 kPa | <5 or >100 kPa |
| Pressure Right | 10–80 kPa | <5 or >100 kPa |
| Moisture | 20–70% | <10% or >85% |
| Wound Status | mild/moderate → high alert | severe → critical alert |

---

## Prototype Hardware Integration

Your hardware device should POST to `/api/sensor-data`:

```python
import requests

data = {
    "patient_id": "PAT001",
    "temperature": sensor.read_temp(),
    "pressure_left": sensor.read_pressure_l(),
    "pressure_right": sensor.read_pressure_r(),
    "moisture": sensor.read_moisture(),
    "wound_status": classifier.classify_wound()
}

response = requests.post("http://YOUR_SERVER_IP:5000/api/sensor-data", json=data)
print(response.json())
```

---

## Production Deployment

### Switch to MySQL
Update `DATABASE_URL` env var and replace sqlite3 with pymysql:
```bash
pip install pymysql
```

### Environment Variables
```bash
SECRET_KEY=your-production-secret
DATABASE_URL=mysql+pymysql://user:pass@host/diafoot
FLASK_DEBUG=False
```

### Gunicorn (Production WSGI)
```bash
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 run:app
```

### Absenteeism Alert Cron Job
Add a cron job to trigger alerts after 9:30 AM daily:
```cron
35 9 * * * curl -X POST http://localhost:5000/api/attendance/generate-absenteeism-alerts
```

---

## Modules Summary

| Module | Description |
|---|---|
| PHC Dashboard | Patient count, active alerts, risk table, charts, reminders |
| Doctor Attendance | Check-in/out, status, history, absenteeism alerts |
| DDHS Central | Multi-PHC overview, attendance bar chart, alert trend chart |
| Prototype Integration | POST /sensor-data → threshold check → alerts |
| Alert Logic | Auto-generate alerts, notify dashboard, severity escalation |
| Patient Management | CRUD, sensor history, risk classification |
