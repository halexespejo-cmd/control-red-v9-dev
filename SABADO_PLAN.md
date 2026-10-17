# 📋 PLAN SÁBADO - Control de Red v9 Launch Prep

**Fecha:** 2026-10-18 (Sábado)  
**Objetivo:** Score 68.9 → 94.85/100  
**Duración:** 09:00-17:00 (8 horas trabajo)

---

## ⏰ TIMELINE COMPLETO

```
08:30 - Despertar + desayuno
09:00 - INICIO trabajo (FASE 1: Railway)
10:30 - FASE 2: Screenshots
12:30 - BREAK almuerzo (30 min)
13:00 - FASE 3: Video demo
15:00 - FASE 4: README update
16:00 - FASE 5: Final touches
17:00 - FIN trabajo
```

**Score objetivo:** 94.85/100 (+25.95 pts)  
**Launch domingo:** READY ✅

---

## 🚀 FASE 1: RAILWAY DEPLOY (09:00-10:30) - CRÍTICO

### **09:00-09:30 | Fix Build Context**

**Problema:** Railway busca `/src` en raíz, código está en `/backend/src`

**Solución A - UI (PREFERIDA):**
```
1. Railway → control-red-v9-dev service
2. Settings → Build
3. Configurar:
   Builder: Dockerfile ✅
   Root Directory: backend    ← AGREGAR
   Dockerfile Path: Dockerfile
4. Deploy
```

**Solución B - railway.toml:**
```bash
cd ~/control-red-v9-dev

cat > railway.toml << 'EOF'
[build]
builder = "DOCKERFILE"
dockerfilePath = "backend/Dockerfile"

[deploy]
startCommand = "uvicorn src.main:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/"
restartPolicyType = "ON_FAILURE"
EOF

git add railway.toml
git commit -m "fix: Railway build context"
git push origin main
```

**Checkpoint 09:30:**
- [ ] Build inicia sin "COPY failed"
- [ ] Logs: "Building from backend/Dockerfile..."

---

### **09:30-10:00 | Monitor Build**

**Watch logs:**
```
Railway → Deployments → Real-time logs

Esperado:
✅ Build successful (6-8 min)
✅ Deploy starting
✅ Health check passed
✅ Service: Active (verde)
```

**Si falla:**
- Port binding → Usar `$PORT` env var
- Database connection → Verificar DATABASE_URL auto-inyectada
- Skill disponible: `railway-troubleshooting`

---

### **10:00-10:30 | Domain + Webhook**

**Generate URL:**
```
1. Settings → Networking → Generate Domain
2. URL: https://control-red-production-XXXX.railway.app
```

**Test:**
```bash
BACKEND_URL="https://control-red-production-XXXX.railway.app"

curl -v $BACKEND_URL/
# Esperado: {"app":"control-red","status":"running"}
```

**Configure webhook:**
```bash
BOT_TOKEN="8712172153:AAEsl_JXbjY4wvM4cerBywmnVvWSkHob2jQ"

curl -X POST "https://api.telegram.org/bot$BOT_TOKEN/setWebhook" \
  -d "url=$BACKEND_URL/api/telegram/webhook"

# Test bot
Telegram → @Mi_red_control_bot → /start
```

**Checkpoint 10:30:**
- [ ] URL generada
- [ ] Health check OK
- [ ] Webhook set
- [ ] Bot responde
- [ ] **✅ RAILWAY FUNCIONANDO (+10.45 pts)**

**Score: 79.35/100**

---

## 📸 FASE 2: SCREENSHOTS (10:30-12:30)

### **10:30-11:00 | Setup**

**Usar v8 (datos reales):**
- PC con v8 corriendo
- Red: BOSQUE_DE_ARCE 2574_5G
- 9 dispositivos reales
- Telegram bot v8 abierto

---

### **11:00-12:00 | Capturar 6 Screenshots**

**Tool:** Win+Shift+S (Windows Screenshot) o Greenshot

**01 - Dashboard Overview** [10 min]
```
Contenido: Dashboard completo, 9 dispositivos
Dimensión: 1920×1080
Nombre: 01-dashboard-overview.png
```

**02 - Device Detail** [10 min]
```
Tap: "Alexa Hab Principal"
Mostrar: IP, MAC, botón "Cortar"
Nombre: 02-device-detail.png
```

**03 - Block Confirmation** [10 min]
```
Tap: Botón "Cortar"
Modal: Confirmación visible
Nombre: 03-block-confirmation.png
⚠️ Cancelar después
```

**04 - Device List Full** [10 min]
```
Scroll: Mostrar todos 9
Nombre: 04-device-list-full.png
```

**05 - Mobile View (opcional)** [10 min]
```
Celular o dev tools responsive
Nombre: 05-mobile-responsive.png
```

**06 - Architecture Diagram** [10 min]
```
Tool: Excalidraw.com

Usuario → Telegram Bot → FastAPI Backend
                              ↓
                     PostgreSQL + Redis
                              ↓
                         LAN Router
                       (ARP Spoofing)

Export: PNG 1920×1080
Nombre: 06-architecture-diagram.png
```

---

### **12:00-12:30 | Upload Imgur**

```
1. imgur.com → New post
2. Upload 6 imágenes
3. Title: "Control de Red v9 - WiFi Management"
4. Privacy: Public (hidden)
5. Copiar URLs

Guardar en:
~/control-red-v9-dev/IMGUR_URLS.txt
```

**Checkpoint 12:30:**
- [ ] 6 screenshots PNG
- [ ] Imgur uploaded
- [ ] URLs guardadas
- [ ] **✅ SCREENSHOTS (+6.2 pts)**

**Score: 85.55/100**

---

## 🍽️ BREAK (12:30-13:00)

Almuerzo + descanso mental

---

## 🎬 FASE 3: VIDEO DEMO (13:00-15:00)

### **13:00-13:30 | Script Review**

**90 segundos:**
```
[00-10s] Hook: "Vecino usa tu WiFi? Fing $9/mes"
[10-20s] Solución: "Control de Red gratis, Telegram"
[20-40s] Demo: Scan → Block device
[40-60s] Tech: "FastAPI, 95% coverage, Railway/Docker"
[60-80s] Valor: "Telegram 900M users, privacidad"
[80-90s] CTA: "Pruébalo, beta welcome, GitHub"
```

---

### **13:30-14:30 | Recording**

**Setup:**
```
1. Cerrar apps innecesarias
2. Browser zoom 110%
3. Full screen (F11)
4. Mock data visible
```

**Tool: OBS Studio**
```
Download: https://obsproject.com/download
Config:
- Output: MP4, x264, 5000 kbps
- Video: 1920×1080, 30fps
- Audio: Desktop (opcional)

Grabar 3-5 takes, elegir mejor
```

---

### **14:30-15:00 | Edit + Upload**

**Edit (Windows Video Editor o CapCut):**
- Trim inicio/fin
- Speed up lentos (1.5x)
- Captions (opcional)

**YouTube Upload:**
```
Title: "Control de Red - WiFi Management from Telegram (Self-Hosted)"
Visibility: Unlisted
URL: https://youtu.be/XXXXXXXXXXX
```

**Checkpoint 15:00:**
- [ ] Video 60-120 seg
- [ ] YouTube uploaded
- [ ] URL copiada
- [ ] **✅ VIDEO (+5.8 pts)**

**Score: 91.35/100**

---

## 📝 FASE 4: README UPDATE (15:00-16:00)

### **15:00-15:30 | Hero Section**

```markdown
# 🌐 Control de Red v9

**WiFi control from Telegram. Free & self-hosted.**

[![Railway](https://railway.app/button.svg)](...)
![Coverage 81.6%](https://img.shields.io/badge/coverage-81.6%25-green)

## 📸 Screenshots

![Dashboard](https://i.imgur.com/abc123.png)
![Detail](https://i.imgur.com/def456.png)

[Imgur Album](https://imgur.com/a/XXXXX)

## 🎬 Demo Video

[![Demo](https://img.youtube.com/vi/XXX/maxresdefault.jpg)](https://youtu.be/XXX)

## ✨ Why Control de Red?

| Feature | This | Fing Premium |
|---------|------|--------------|
| Price | Free | $8.99/month |
| Install | Telegram | Separate app |
| Privacy | Your server | Cloud |
```

Replace: URLs Imgur + YouTube

---

### **15:30-16:00 | Commit + Push**

```bash
echo "# Media URLs
Screenshots: https://imgur.com/a/XXXXX
Video: https://youtu.be/XXXXXXXXXXX
" > MEDIA_URLS.md

git add README.md MEDIA_URLS.md
git commit -m "docs: Add hero with screenshots and video"
git push origin main
```

**Checkpoint 16:00:**
- [ ] README visual
- [ ] Commit signed
- [ ] GitHub updated
- [ ] **✅ README HERO (+3.5 pts)**

**Score: 94.85/100** 🚀

---

## 🎯 FASE 5: FINAL TOUCHES (16:00-17:00)

### **16:00-16:30 | Update Launch Materials**

**Edit REDDIT_POST.md:**
```markdown
## Live Demo
Screenshots: [Imgur](...)
Video: [YouTube](...)
Deploy: Railway button
```

**Edit HN_SUBMISSION.md:**
```markdown
Links:
- Demo video: https://youtu.be/...
- Screenshots: https://imgur.com/...
- GitHub: https://github.com/...
```

---

### **16:30-17:00 | Final Sync**

```bash
git add REDDIT_POST.md HN_SUBMISSION.md
git commit -m "docs: Update launch posts with media"
git push origin main

# OneDrive sync
cp REDDIT_POST.md HN_SUBMISSION.md MEDIA_URLS.md \
   "/mnt/c/Users/halex/OneDrive - Operadora Stratexia Quantum/Developer Projects/Control de Red v9/marketing/"
```

**Checkpoint 17:00:**
- [ ] Launch materials updated
- [ ] OneDrive synced
- [ ] All committed
- [ ] **✅ LAUNCH READY**

---

## ✅ CHECKLIST DÍA COMPLETO

```
SÁBADO:

09:00 [ ] Railway: Root Directory = backend
09:30 [ ] Monitor build success
10:00 [ ] Generate domain
10:15 [ ] Test health check
10:20 [ ] Configure webhook
10:25 [ ] Test bot /start
      [ ] ✅ RAILWAY (+10.45 pts) → 79.35/100

10:30 [ ] Setup v8 for screenshots
11:00 [ ] Screenshot 1: Dashboard
11:10 [ ] Screenshot 2: Device detail
11:20 [ ] Screenshot 3: Block confirmation
11:30 [ ] Screenshot 4: Device list
11:40 [ ] Screenshot 5: Mobile (opcional)
11:50 [ ] Screenshot 6: Architecture
12:00 [ ] Upload Imgur + URLs
      [ ] ✅ SCREENSHOTS (+6.2 pts) → 85.55/100

12:30 [ ] BREAK - Almuerzo

13:00 [ ] Review script 90 seg
13:30 [ ] Setup OBS Studio
13:45 [ ] Record 3-5 takes
14:30 [ ] Edit (trim, speed)
14:50 [ ] Upload YouTube (Unlisted)
15:00 [ ] Copy URL
      [ ] ✅ VIDEO (+5.8 pts) → 91.35/100

15:00 [ ] README: Hero section
15:15 [ ] Embed screenshots
15:20 [ ] Embed video
15:25 [ ] Comparison table
15:30 [ ] Git commit + push
      [ ] ✅ README (+3.5 pts) → 94.85/100

16:00 [ ] Update REDDIT_POST.md
16:10 [ ] Update HN_SUBMISSION.md
16:20 [ ] Sync OneDrive
16:30 [ ] Final commit
17:00 [ ] ✅ READY FOR LAUNCH

SCORE FINAL: 94.85/100
LAUNCH DOMINGO: ✅ LISTO
```

---

## 🚨 FALLBACK PLANS

### **Railway falla (>90 min debug):**

**Plan B: Render.com** (15 min)
```
1. render.com → New Web Service
2. GitHub: control-red-v9-dev
3. Auto-detects Dockerfile ✅
4. Root: backend
5. Env vars
6. Deploy → Funciona
```

**Plan C: Fly.io** (20 min)

**Plan D: Use v8 for demo**
- Screenshots con v8 ✅
- v9 cloud post-launch

### **Video difícil (>2 horas):**
- Skip video
- Solo screenshots (suficiente)
- Score: 88.75/100 (aún excelente)

### **Tiempo corto:**
- Reducir screenshots a 4
- Skip mobile view
- Skip architecture diagram

---

## 📊 SCORE PROGRESSION

```
Viernes fin:       68.9/100
+ Railway:        +10.45 → 79.35
+ Screenshots:     +6.20 → 85.55
+ Video:           +5.80 → 91.35
+ README:          +3.50 → 94.85
────────────────────────────
TOTAL SÁBADO:     94.85/100

Launch Ready: ✅
Quality: Production ✅
Timeline: On track ✅
```

---

## 💡 TIPS EJECUCIÓN

**Railway:**
- Skill: `railway-troubleshooting`
- Si >30min debug → Render.com
- No perder tiempo en debugging infinito

**Screenshots:**
- v8 datos reales > v9 mock
- Calidad > cantidad (4 buenos > 6 mediocres)
- Imgur hidden public (URLs funcionan, no searchable)

**Video:**
- 3 takes suficiente
- Audio opcional (captions compensan)
- <100 seg okay

**README:**
- Embed directo Imgur (no dowload)
- YouTube embed con preview image
- Comparison table = selling point

---

## 🎯 OBJETIVOS DOMINGO LAUNCH

```
08:00 - Reddit r/homelab post
10:00 - Hacker News Show HN
12:00-20:00 - Engagement activo

Metrics:
- 20+ upvotes Reddit
- 50+ points HN
- 5+ GitHub stars
- 10+ demo tries

Success = Product-Market Fit validation
```

---

**Plan guardado:** `~/control-red-v9-dev/SABADO_PLAN.md`  
**Última actualización:** 2026-10-17 17:00

---

¡Éxito mañana! 🚀
