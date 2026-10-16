# 🚂 RAILWAY SETUP - INSTRUCCIONES

## PASO 1: Crear Repo GitHub v9

**En tu navegador:**

1. Ve a: https://github.com/new
2. **Repository name:** `control-red-v9-dev`
3. **Description:** `🌐 WiFi network control via Telegram Mini App - FastAPI + PostgreSQL + Redis`
4. **Visibility:** Public ✅
5. **NO** marcar "Initialize with README" (ya tienes código)
6. Click **"Create repository"**

---

## PASO 2: Actualizar Remote Local

**En tu terminal WSL:**

```bash
cd ~/control-red-v9-dev

# Ver remote actual (apunta a v8)
git remote -v

# Actualizar remote a v9
git remote set-url origin https://github.com/halexespejo-cmd/control-red-v9-dev.git

# Verificar
git remote -v

# Push a nuevo repo
git push -u origin main
```

**Expected output:**
```
Enumerating objects: 500, done.
Counting objects: 100% (500/500), done.
...
To https://github.com/halexespejo-cmd/control-red-v9-dev.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

---

## PASO 3: Railway Project Setup

### **3.1: Conectar Railway a GitHub**

**En navegador:**

1. Ve a: https://railway.app/new
2. Click **"Deploy from GitHub repo"**
3. Si primera vez:
   - Click **"Configure GitHub App"**
   - Autorizar Railway acceso a repos
   - Seleccionar repos: **"All repositories"** o solo `control-red-v9-dev`
4. Selecciona repo: **halexespejo-cmd/control-red-v9-dev**
5. Railway detecta `Dockerfile` ✅

### **3.2: Agregar PostgreSQL**

1. En tu proyecto Railway, click **"+ New"**
2. Selecciona **"Database"**
3. Click **"Add PostgreSQL"**
4. Railway automáticamente:
   - Provisiona database
   - Genera `DATABASE_URL`
   - Inyecta variable en tu service

### **3.3: Agregar Redis**

1. Click **"+ New"** nuevamente
2. Selecciona **"Database"**
3. Click **"Add Redis"**  
4. Railway automáticamente:
   - Provisiona Redis
   - Genera `REDIS_URL`
   - Inyecta variable en tu service

### **3.4: Configurar Variables Env**

1. Click en tu service "control-red-v9-dev"
2. Pestaña **"Variables"**
3. Click **"+ New Variable"**

**Agregar manualmente:**

```env
APP_NAME=control-red
DEBUG=false
LOG_LEVEL=INFO
VERSION=9.0.0-dev.13

# Secret keys (usar Railway's Random String generator)
SECRET_KEY=<click "Generate" → Random String>
JWT_SECRET_KEY=<click "Generate" → Random String>

# Telegram Bot (tu token actual)
TELEGRAM_BOT_TOKEN=8712172153:AAEsl_JXbjY4wvM4cerBywmnVvWSkHob2jQ
```

**Railways auto-inyecta estas (NO agregar manual):**
- ✅ `DATABASE_URL`
- ✅ `REDIS_URL`
- ✅ `PORT`

### **3.5: Deploy**

1. Railway auto-deploys cuando detecta repo
2. Ver logs: Click service → **"Deployments"** → **"View Logs"**

**Expected logs:**
```
Building...
[+] Building 120.5s
...
✅ Build successful
Starting...
INFO: Started server process [1]
INFO: Waiting for application startup.
🚀 Starting control-red v9.0.0-dev.13
✅ Database initialized
✅ Redis cache initialized
✅ ARP Blocker initialized
INFO: Application startup complete.
```

3. Cuando logs muestren "Application startup complete":
   - Click pestaña **"Settings"**
   - Scroll a **"Domains"**
   - Click **"Generate Domain"**
   - Copia tu URL: `https://control-red-production-xxxx.up.railway.app`

---

## PASO 4: Verificación Health Check

**En tu terminal:**

```bash
# Guardar URL
export RAILWAY_URL="https://control-red-production-xxxx.up.railway.app"

# Health check
curl $RAILWAY_URL/

# Expected output:
# {
#   "app": "control-red",
#   "version": "9.0.0-dev.13",
#   "status": "running",
#   "debug": false
# }

# Test API endpoint
curl $RAILWAY_URL/api/devices/

# Expected:
# [] (array vacío ok, significa DB funcionando)
```

---

## PASO 5: Test Completo

```bash
# 1. Register nuevo usuario
curl -X POST $RAILWAY_URL/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "Test123456!",
    "full_name": "Test User"
  }'

# Expected: 
# {
#   "id": 1,
#   "email": "test@example.com",
#   "full_name": "Test User",
#   "is_active": true,
#   "access_token": "eyJ...",
#   "token_type": "bearer"
# }

# 2. Login
TOKEN=$(curl -X POST $RAILWAY_URL/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test@example.com",
    "password": "Test123456!"
  }' | jq -r .access_token)

echo "Token: $TOKEN"

# 3. Test autenticado
curl -H "Authorization: Bearer $TOKEN" \
  $RAILWAY_URL/api/devices/

# Expected:
# [] (ok)
```

---

## PASO 6: Configurar Telegram Webhook

```bash
# Con tu URL Railway
export BOT_TOKEN="8712172153:AAEsl_JXbjY4wvM4cerBywmnVvWSkHob2jQ"
export WEBHOOK_URL="$RAILWAY_URL/api/telegram/webhook"

# Configurar webhook
curl -X POST https://api.telegram.org/bot$BOT_TOKEN/setWebhook \
  -H "Content-Type: application/json" \
  -d "{\"url\": \"$WEBHOOK_URL\"}"

# Expected:
# {
#   "ok": true,
#   "result": true,
#   "description": "Webhook was set"
# }

# Verificar webhook
curl https://api.telegram.org/bot$BOT_TOKEN/getWebhookInfo

# Expected:
# {
#   "ok": true,
#   "result": {
#     "url": "https://control-red-production-xxxx.railway.app/api/telegram/webhook",
#     "has_custom_certificate": false,
#     "pending_update_count": 0,
#     "max_connections": 40
#   }
# }
```

---

## PASO 7: Test Bot Live

**En Telegram (tu celular):**

1. Abre Telegram
2. Busca: `@Mi_red_control_bot`
3. Tap **START**
4. Envía: `/start`

**Expected:**
```
🌐 Control de Red - Bienvenido

Controla tu WiFi desde Telegram en 3 pasos:

1️⃣ Conecta tu backend
   → Railway (1-click) o Docker local

2️⃣ Escanea tu red
   → Descubre todos los dispositivos

3️⃣ Bloquea/desbloquea
   → Control total en segundos

¿Listo para empezar? 👇

[🚀 Setup Backend] [📖 Ver Docs]
[💬 Ayuda]        [ℹ️ Acerca de]
```

5. Tap botón **"💬 Ayuda"**

**Expected:**
```
📚 Comandos Disponibles

Setup:
/start - Mensaje de bienvenida
/setup - Guía configuración backend
...
```

---

## ✅ CHECKLIST POST-DEPLOY

- [ ] Railway service deployed (verde)
- [ ] PostgreSQL connected (ver logs)
- [ ] Redis connected (ver logs)
- [ ] Health check `/` retorna 200
- [ ] API `/api/devices/` retorna array
- [ ] Auth `/api/auth/register` funciona
- [ ] Webhook configurado (getWebhookInfo ok)
- [ ] Bot responde `/start` en Telegram
- [ ] Bot responde `/help`
- [ ] Inline buttons funcionan

---

## 🐛 TROUBLESHOOTING

### **Error: "Build failed"**

Ver logs Railway → buscar:
- Missing dependencies → agregar a `requirements.txt`
- Python version → cambiar Dockerfile `FROM python:3.11-slim`

### **Error: "Database connection failed"**

```bash
# Verificar variable existe
railway variables list

# Debe mostrar DATABASE_URL
# Si no: Re-link PostgreSQL service
```

### **Error: "Bot no responde"**

```bash
# Verificar webhook set
curl https://api.telegram.org/bot$BOT_TOKEN/getWebhookInfo

# Si "url" está vacío → Re-setear webhook
# Si "url" correcto → Ver Railway logs buscar errors
```

### **Error: "ARP Blocker not initialized"**

**Normal en Railway** - containers no tienen `CAP_NET_ADMIN`

Ver logs debe decir:
```
⚠️  ARP Blocker initialization failed: Permission denied (run with sudo for real blocking)
```

**Es OK** - el resto funciona. ARP solo en local con sudo.

---

## 📊 ESTIMADO TIEMPO

- Crear repo GitHub: 2 min
- Actualizar remote local: 2 min
- Railway project setup: 5 min
- Add PostgreSQL + Redis: 2 min
- Configure variables: 3 min
- Deploy + wait build: 5 min
- Test health checks: 3 min
- Configure webhook: 2 min
- Test bot live: 2 min

**TOTAL: ~25 minutos**

---

## 🎯 SIGUIENTE PASO

Una vez todo ✅ → Actualizar `README.md` con:
- Railway deploy button
- Tu URL real
- Bot instructions

¡Empieza ahora! 🚀
