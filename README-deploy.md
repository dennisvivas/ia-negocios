# Deploy en dennisvivas.com — Guía paso a paso

Stack: **Vercel** (hosting estático) + **Cloudflare** (edge security, CDN, WAF)

---

## 1. Conectar el repo a Vercel

1. Ve a [vercel.com](https://vercel.com) e inicia sesión con tu cuenta de GitHub
2. Clic en **Add New → Project**
3. Busca y selecciona el repo `dennisvivas/presentacion-ia-negocios`
4. En la pantalla de configuración:
   - **Framework Preset**: `Other`
   - **Root Directory**: `/` (raíz, dejarlo en blanco)
   - **Build Command**: dejar en blanco (sitio estático)
   - **Output Directory**: dejar en blanco
5. Clic en **Deploy** — el primer deploy tarda ~30 segundos

El sitio queda live en una URL tipo `presentacion-ia-negocios.vercel.app`.

---

## 2. Agregar dominio custom en Vercel

1. En el dashboard del proyecto → **Settings → Domains**
2. Escribe `dennisvivas.com` y clic en **Add**
3. Repite con `www.dennisvivas.com`
4. Vercel te mostrará los registros DNS que necesitas configurar (ver paso 3)

---

## 3. Configurar DNS en Cloudflare

> Prerrequisito: tu dominio `dennisvivas.com` ya debe estar en Cloudflare.
> Si no lo está, transfiere los nameservers de tu registrador a Cloudflare
> antes de continuar.

### 3a. Registros DNS

En Cloudflare → **DNS → Records**, agrega o edita:

| Tipo  | Nombre | Valor                  | Proxy       |
|-------|--------|------------------------|-------------|
| A     | `@`    | `76.76.21.21`          | ☁️ Proxied  |
| CNAME | `www`  | `cname.vercel-dns.com` | ☁️ Proxied  |

> La nube naranja (Proxied) activa el CDN y el WAF de Cloudflare.

### 3b. SSL/TLS

En Cloudflare → **SSL/TLS → Overview**:
- Modo: **Full (strict)**

En **SSL/TLS → Edge Certificates**:
- ✅ Always Use HTTPS
- ✅ HTTP Strict Transport Security (HSTS)
  - Max Age: 12 months
  - Include subdomains: activado
  - Preload: activado
- ✅ Minimum TLS Version: TLS 1.2
- ✅ Opportunistic Encryption

### 3c. Performance

En **Speed → Optimization**:
- ✅ Brotli compression
- ✅ Early Hints

### 3d. WAF (Web Application Firewall)

En **Security → WAF**:
- El plan Free incluye las reglas gestionadas de Cloudflare — están activas por defecto
- En **Security → Settings**:
  - Security Level: **Medium**
  - Bot Fight Mode: ✅ activado

---

## 4. Verificar el deploy

Una vez propagado el DNS (puede tardar hasta 5 minutos con Cloudflare):

```
# Verifica headers de seguridad
curl -I https://dennisvivas.com

# Debe mostrar:
# strict-transport-security: max-age=31536000; includeSubDomains; preload
# x-content-type-options: nosniff
# x-frame-options: DENY
# content-security-policy: ...
```

También puedes usar [securityheaders.com](https://securityheaders.com/?q=dennisvivas.com) para un reporte completo.

---

## URLs del sitio

| Ruta | Contenido |
|------|-----------|
| `dennisvivas.com` | Presentación interactiva (navegación con ←→) |
| `dennisvivas.com/print` | Versión para exportar a PDF |

---

## Actualizaciones futuras

Cualquier `git push` a `main` dispara un redeploy automático en Vercel (< 30 segundos).
