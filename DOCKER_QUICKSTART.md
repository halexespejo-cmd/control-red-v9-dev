# 🐳 DOCKER QUICKSTART - Guía de Uso

**Setup completo en 1 comando:** PostgreSQL + Redis + Backend

---

## 🚀 Quick Start (3 minutos)

### **Opción 1: Script automático (Recomendado)**

```bash
# Desde la raíz del proyecto
./scripts/quickstart.sh
```

**Eso es todo!** El script:
- ✅ Verifica Docker instalado
- ✅ Genera `.env` con secrets seguros
- ✅ Levanta PostgreSQL + Redis + Backend
- ✅ Espera health checks
- ✅ Te da la URL: `http://localhost:8000`

---

### **Opción 2: Manual (si script falla)**

```bash
# 1. Generar .env
cat > backend/.env << 'EOF'
APP_NAME=control-red
DEBUG=true
DATABASE_URL=postgresql+asyncpg://control_red_user:control_red_pass@postgres:5432/control_red
REDIS_URL=redis://redis:6379/0
SECRET_KEY=change-me-in-production
JWT_SECRET_KEY=change-me-in-production
EOF

# 2. Levantar servicios
docker-compose up -d

# 3. Ver logs
docker-compose logs -f
```

---

## ✅ Verificación

### **Health check:**
```bash
curl http://localhost:8000/
```

**Expected output:**
```json
{
  "app": "control-red",
  "version": "9.0.0-dev.13",
  "status": "running",
  "debug": true
}
```

### **Test API:**
```bash
# Lista dispositivos (vacío ok)
curl http://localhost:8000/api/devices/

# Expected: []
```

### **Swagger docs:**
Abre navegador: http://localhost:8000/docs

---

## 📊 Ver Logs

```bash
# Todos los servicios
docker-compose logs -f

# Solo backend
docker-compose logs -f backend

# Solo PostgreSQL
docker-compose logs -f postgres

# Solo Redis
docker-compose logs -f redis

# Últimas 50 líneas
docker-compose logs --tail=50 backend
```

---

## 🔧 Comandos Útiles

### **Reiniciar servicios:**
```bash
docker-compose restart
```

### **Reiniciar solo backend:**
```bash
docker-compose restart backend
```

### **Detener todo:**
```bash
docker-compose down
```

### **Detener y eliminar datos:**
```bash
# ⚠️  CUIDADO: Borra base de datos PostgreSQL
docker-compose down -v
```

### **Rebuild (después de cambios código):**
```bash
docker-compose up -d --build
```

### **Ver servicios corriendo:**
```bash
docker-compose ps
```

### **Acceder a contenedor:**
```bash
# Backend shell
docker exec -it control-red-backend bash

# PostgreSQL psql
docker exec -it control-red-postgres psql -U control_red_user -d control_red

# Redis CLI
docker exec -it control-red-redis redis-cli
```

---

## 🤖 Configurar Telegram Bot

### **1. Agregar token a .env:**

Edita `backend/.env`:
```bash
TELEGRAM_BOT_TOKEN=8712172153:AAEsl_JXbjY4wvM4cerBywmnVvWSkHob2jQ
```

### **2. Reiniciar backend:**
```bash
docker-compose restart backend
```

### **3. Verificar logs:**
```bash
docker-compose logs backend | grep -i telegram
```

Deberías ver:
```
INFO: ✅ Telegram bot handler initialized
```

### **4. Test bot local (requiere túnel):**

**Opción A: ngrok**
```bash
# Terminal 1: Levantar túnel
ngrok http 8000

# Terminal 2: Configurar webhook
curl -X POST https://api.telegram.org/bot<TOKEN>/setWebhook \
  -d url=https://abc123.ngrok.io/api/telegram/webhook

# Test en Telegram
# @Mi_red_control_bot → /start
```

**Opción B: localhost.run**
```bash
ssh -R 80:localhost:8000 localhost.run
```

---

## 🐛 Troubleshooting

### **Error: "Cannot connect to Docker daemon"**

**Causa:** Docker no está corriendo

**Solución:**
```bash
# Linux
sudo systemctl start docker

# macOS/Windows
# Abrir Docker Desktop app
```

---

### **Error: "port 8000 already in use"**

**Causa:** Otro proceso usando puerto 8000

**Solución:**
```bash
# Ver qué usa el puerto
sudo lsof -i :8000

# Matar proceso
kill -9 <PID>

# O cambiar puerto en docker-compose.yml:
# ports:
#   - "8001:8000"  # Host:Container
```

---

### **Error: "pg_isready: timeout"**

**Causa:** PostgreSQL no inició

**Solución:**
```bash
# Ver logs
docker-compose logs postgres

# Reintentar
docker-compose restart postgres

# Verificar health
docker-compose ps
```

---

### **Backend crashea con "Connection refused"**

**Causa:** Backend intentó conectar antes de que PostgreSQL esté ready

**Solución:**
```bash
# docker-compose.yml ya tiene depends_on + healthcheck
# Pero puedes agregar delay manual:

docker-compose up -d postgres redis
sleep 10
docker-compose up -d backend
```

---

### **ARP spoofing no funciona**

**Causa:** Contenedor no tiene permisos `NET_ADMIN`

**Ya configurado** en `docker-compose.yml`:
```yaml
cap_add:
  - NET_ADMIN
  - NET_RAW
```

**Verificar:**
```bash
docker inspect control-red-backend | grep -A5 CapAdd
```

**Debería mostrar:** `"NET_ADMIN"`, `"NET_RAW"`

---

## 📂 Estructura Volúmenes

```
docker-compose.yml define 2 volúmenes:

postgres-data/  → Base de datos persistente
redis-data/     → Cache Redis persistente

Ubicación física:
  Linux: /var/lib/docker/volumes/
  macOS: ~/Library/Containers/com.docker.docker/Data/
  Windows: \\wsl$\docker-desktop-data\data\docker\volumes\
```

**Backup base de datos:**
```bash
docker exec control-red-postgres pg_dump \
  -U control_red_user control_red > backup.sql
```

**Restore:**
```bash
docker exec -i control-red-postgres psql \
  -U control_red_user control_red < backup.sql
```

---

## ⚙️ Variables de Entorno

**En `backend/.env`:**

| Variable | Descripción | Default | Required |
|----------|-------------|---------|----------|
| `APP_NAME` | Nombre app | `control-red` | ✅ |
| `DEBUG` | Modo debug | `true` | ✅ |
| `DATABASE_URL` | PostgreSQL connection | (ver compose) | ✅ |
| `REDIS_URL` | Redis connection | `redis://redis:6379/0` | ✅ |
| `SECRET_KEY` | Session encryption | (generar) | ✅ |
| `JWT_SECRET_KEY` | JWT tokens | (generar) | ✅ |
| `TELEGRAM_BOT_TOKEN` | Bot Telegram | - | ❌ |
| `SENTRY_DSN` | Error tracking | - | ❌ |
| `LOG_LEVEL` | Verbosity | `INFO` | ❌ |

**Generar secrets seguros:**
```bash
openssl rand -hex 32
# o
python3 -c "import secrets; print(secrets.token_hex(32))"
```

---

## 🚀 Deploy a Producción

**Este setup es para desarrollo local.**

Para producción, usa:
- **Railway:** Ver `RAILWAY_DEPLOY.md`
- **VPS:** Ver docs/deployment/vps.md (TODO)
- **Heroku:** No recomendado (ARP spoofing no funciona)

---

## 📝 Diferencias vs Railway

| Aspecto | Docker Local | Railway |
|---------|--------------|---------|
| **Setup** | 3 min manual | 1-click |
| **Database** | Local PostgreSQL | Managed PostgreSQL |
| **HTTPS** | No (localhost) | Sí automático |
| **Uptime** | Mientras PC ON | 99.9% SLA |
| **Cost** | $0 | ~$5-10/mes |
| **ARP blocking** | ✅ Funciona | ❌ No permissions |
| **Telegram webhook** | Requiere túnel | Directo |

**Conclusión:** Local para desarrollo, Railway para producción.

---

## ✅ Checklist Post-Setup

- [ ] Backend responde en `http://localhost:8000`
- [ ] Swagger docs carga en `/docs`
- [ ] PostgreSQL health green (`docker-compose ps`)
- [ ] Redis health green
- [ ] Logs backend sin errores críticos
- [ ] Can register user: `POST /api/auth/register`
- [ ] Can scan devices: `POST /api/devices/scan`
- [ ] Telegram bot token configurado (si usas)

**Si todo ✅ → Listo para desarrollo!**

---

**Última actualización:** 16 Octubre 2026  
**Mantenedor:** @halexespejo-cmd
