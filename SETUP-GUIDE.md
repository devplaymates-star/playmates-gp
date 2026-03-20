# Guida Setup — Playmates Grand Prix Lead Automation

## 1. Google Sheet (CRM)

Crea un nuovo Google Sheet e copia l'ID dalla URL (`https://docs.google.com/spreadsheets/d/TUO_ID/edit`).

### Tab "Leads" — colonne:
| Timestamp | Nome | Cognome | Email | Telefono | Pacchetto | Persone | Messaggio | Lingua | Source | UTM_Source | UTM_Medium | UTM_Campaign | Page_URL |

### Tab "Exit Popup" — colonne:
| Timestamp | Email | Source | Lingua | Stato |

### Tab "WhatsApp Clicks" — colonne:
| Timestamp | Lingua | UTM_Source | UTM_Medium | UTM_Campaign | Referrer | Page_URL |

---

## 2. CallMeBot (Notifiche WhatsApp gratuite)

1. Salva in rubrica il numero **+34 644 71 86 39** (CallMeBot)
2. Invia il messaggio: `I allow callmebot to send me messages`
3. Riceverai una API key — annotala
4. Testa: `https://api.callmebot.com/whatsapp.php?phone=TUO_NUMERO&text=Test&apikey=TUA_APIKEY`

---

## 3. Importare il workflow n8n

1. Apri n8n → **Workflows** → **Import from File**
2. Seleziona `n8n-workflow.json`
3. Il workflow apparirà con tutti i nodi già collegati

---

## 4. Placeholder da sostituire

### Nel file `index.html`:
| Placeholder | Dove | Descrizione |
|------------|------|-------------|
| `WEBHOOK_URL_FORM` | funzione handleForm | URL webhook n8n /playmates-lead |
| `WEBHOOK_URL_EXIT` | funzione exitSubmit | URL webhook n8n /playmates-exit |
| `WEBHOOK_URL_WA_CLICK` | funzione waClick | URL webhook n8n /playmates-wa-click |
| `WHATSAPP_NUMBER` | link wa.me nel bottone | Numero WhatsApp (es: 393331234567) |

### Nel workflow n8n:
| Placeholder | Nodi | Descrizione |
|------------|------|-------------|
| `TUO_GOOGLE_SHEET_ID` | Tutti i nodi Google Sheets | ID del Google Sheet |
| `TUO_CREDENTIAL_ID` | Tutti i nodi Google Sheets | Credenziale Google Sheets in n8n |
| `TUO_SMTP_ID` | Tutti i nodi Email | Credenziale SMTP in n8n |
| `CALLMEBOT_PHONE` | Nodi HTTP Request CallMeBot | Tuo numero telefono |
| `CALLMEBOT_APIKEY` | Nodi HTTP Request CallMeBot | API key CallMeBot |
| `LINK_DOSSIER` | Email dossier | URL del PDF dossier evento |
| `LINK_CALENDLY` | Email last chance | URL per prenotare chiamata |
| `LINK_SITO` | Email | URL del sito live |

---

## 5. Credenziali n8n

### Google Sheets:
1. n8n → **Credentials** → **Add Credential** → **Google Sheets OAuth2**
2. Segui la procedura di autenticazione Google
3. Copia l'ID credenziale e sostituisci `TUO_CREDENTIAL_ID` nei nodi

### SMTP (per le email):
1. n8n → **Credentials** → **Add Credential** → **SMTP**
2. Configura: host, port, user, password del tuo provider email
3. Copia l'ID e sostituisci `TUO_SMTP_ID` nei nodi

---

## 6. Attivare il workflow

1. Apri il workflow importato in n8n
2. Clicca **Activate** (toggle in alto a destra)
3. Vai su ogni nodo Webhook e copia la **Production URL**
4. Incolla gli URL nei placeholder del file `index.html`

---

## 7. Applicare le modifiche al sito

```bash
cd /Users/srperezalex/playmates-gp
# Sostituisci i placeholder (esempio):
sed -i '' 's|WEBHOOK_URL_FORM|https://tuo-n8n.com/webhook/abc123|g' index.html
sed -i '' 's|WEBHOOK_URL_EXIT|https://tuo-n8n.com/webhook/def456|g' index.html
sed -i '' 's|WEBHOOK_URL_WA_CLICK|https://tuo-n8n.com/webhook/ghi789|g' index.html
sed -i '' 's|WHATSAPP_NUMBER|393331234567|g' index.html

# Push live
git add -A && git commit -m "Add webhook URLs" && git push
```

---

## 8. Meta Pixel e TikTok Pixel

Aggiungi questi snippet dentro `<head>` nel file `index.html`:

### Meta Pixel:
```html
<script>
!function(f,b,e,v,n,t,s){if(f.fbq)return;n=f.fbq=function(){n.callMethod?
n.callMethod.apply(n,arguments):n.queue.push(arguments)};if(!f._fbq)f._fbq=n;
n.push=n;n.loaded=!0;n.version='2.0';n.queue=[];t=b.createElement(e);t.async=!0;
t.src=v;s=b.getElementsByTagName(e)[0];s.parentNode.insertBefore(t,s)}(window,
document,'script','https://connect.facebook.net/en_US/fbevents.js');
fbq('init', 'META_PIXEL_ID');
fbq('track', 'PageView');
</script>
```

### TikTok Pixel:
```html
<script>
!function(w,d,t){w.TiktokAnalyticsObject=t;var ttq=w[t]=w[t]||[];
ttq.methods=["page","track","identify","instances","debug","on","off","once","ready","alias","group","enableCookie","disableCookie"];
ttq.setAndDefer=function(t,e){t[e]=function(){t.push([e].concat(Array.prototype.slice.call(arguments,0)))}};
for(var i=0;i<ttq.methods.length;i++)ttq.setAndDefer(ttq,ttq.methods[i]);
ttq.instance=function(t){for(var e=ttq._i[t]||[],n=0;n<ttq.methods.length;n++)ttq.setAndDefer(e,ttq.methods[n]);return e};
ttq.load=function(e,n){var i="https://analytics.tiktok.com/i18n/pixel/events.js";
ttq._i=ttq._i||{};ttq._i[e]=[];ttq._i[e]._u=i;ttq._o=ttq._o||{};ttq._o[e]=n||{};
var o=document.createElement("script");o.type="text/javascript";o.async=!0;o.src=i+"?sdkid="+e+"&lib="+t;
var a=document.getElementsByTagName("script")[0];a.parentNode.insertBefore(o,a)};
ttq.load('TIKTOK_PIXEL_ID');
ttq.page();
}(window,document,'ttq');
</script>
```

---

## 9. Testing

Testa in TUTTE le lingue (`?lang=it`, `?lang=en`, `?lang=fr`, `?lang=es`):

- [ ] **Form**: compila e invia → controlla Google Sheet + email conferma + notifica WhatsApp
- [ ] **Exit Popup**: muovi mouse fuori dalla pagina (desktop) / aspetta 45s (mobile) → compila email → controlla Sheet
- [ ] **WhatsApp Button**: clicca → controlla Sheet "WhatsApp Clicks" + apertura WhatsApp
- [ ] **Sequenza email**: dopo 1h arriva dossier, dopo 24h reminder WhatsApp, dopo 48h urgency email, dopo 5gg last chance
- [ ] **Pixel**: usa Meta Pixel Helper e TikTok Pixel Helper per verificare gli eventi

---

## 10. Struttura UTM per le ads

Usa questa struttura per le ads su Meta e TikTok:

```
https://devplaymates-star.github.io/playmates-gp/?lang=it&utm_source=meta&utm_medium=paid&utm_campaign=italia_gp2026&utm_content=video_hero
```

### Parametri:
| Parametro | Valori esempio |
|-----------|---------------|
| `lang` | `it`, `en`, `fr`, `es` — manda il traffico nella lingua giusta |
| `utm_source` | `meta`, `tiktok`, `google`, `instagram` |
| `utm_medium` | `paid`, `organic`, `email`, `referral` |
| `utm_campaign` | `italia_gp2026`, `france_gp2026`, `spain_gp2026`, `uk_gp2026` |
| `utm_content` | `video_hero`, `carousel_venue`, `story_countdown` |

### Esempi per paese:
- **Italia**: `?lang=it&utm_source=meta&utm_campaign=italia_gp2026`
- **Francia**: `?lang=fr&utm_source=meta&utm_campaign=france_gp2026`
- **Spagna/LATAM**: `?lang=es&utm_source=tiktok&utm_campaign=spain_gp2026`
- **UK/Mondo**: `?lang=en&utm_source=meta&utm_campaign=global_gp2026`

---

## 11. Sequenza Follow-up

| Tempo | Azione | Destinatario | Canale | Lingua |
|-------|--------|-------------|--------|--------|
| 0 min | Conferma ricezione | Lead | Email | Lingua del lead |
| 0 min | Notifica nuovo lead | Te | WhatsApp | Italiano |
| 1 ora | Dossier foto 2025 | Lead | Email | Lingua del lead |
| 24 ore | Reminder chiamata | Te | WhatsApp | Italiano |
| 48 ore | Email urgency FOMO | Lead | Email | Lingua del lead |
| 5 giorni | Email last chance | Lead | Email | Lingua del lead |
