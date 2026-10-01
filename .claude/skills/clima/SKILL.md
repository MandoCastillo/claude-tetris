---
name: clima
description: Consulta el clima actual o el pronóstico de la ubicación local (detectada por IP) o de una ciudad dada, usando wttr.in sin API key. Usar cuando el usuario pida "clima", "tiempo", "temperatura", "pronóstico", "¿va a llover?" o invoque /clima.
argument-hint: "[ciudad] [pronostico]"
allowed-tools: Bash(curl:*)
---

# Clima

Obtén el clima con `curl` contra wttr.in. Sin argumentos usa la ubicación local (por IP).
Si el usuario da una ciudad, ponla en la URL con espacios como `+` (ej. `Ciudad+de+Mexico`).

## Clima actual

```bash
curl -s --max-time 10 "wttr.in/CIUDAD?format=%l:+%c+%t+(sensación+%f),+humedad+%h,+viento+%w,+lluvia+%p&lang=es&m"
```

Deja `CIUDAD` vacío para la ubicación local: `wttr.in/?format=...`.

## Pronóstico (hoy + 2 días)

Si piden pronóstico, "mañana" o "¿va a llover?":

```bash
curl -s --max-time 10 "wttr.in/CIUDAD?format=j1&lang=es" | python3 -c "
import json,sys
d=json.load(sys.stdin)
for w in d['weather']:
    h=w['hourly'][4]
    print(w['date'], f\"min {w['mintempC']}°C / max {w['maxtempC']}°C,\", h['lang_es'][0]['value'], f\"lluvia {max(int(x['chanceofrain']) for x in w['hourly'])}%\")
"
```

## Respuesta

- Responde en español, en 1–3 líneas: ubicación, estado, temperatura y lo relevante a la pregunta.
- Si `curl` falla o devuelve "Unknown location", dilo y pide una ciudad concreta.
