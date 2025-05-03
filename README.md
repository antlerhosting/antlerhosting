## System Components
### 1. Vue Frontend (Marketing & Order Entry)
- A **static SPA** compiled with Vue CLI, served via Nginx for global CDN caching and minimal server load ([Vue CLI](https://cli.vuejs.org/guide/deployment?utm_source=chatgpt.com "Deployment - Vue CLI")).
- Implements SEO best practices (meta tags via `vue-meta`, pre‐rendered index.html) for maximum discoverability ([ASPER BROTHERS](https://asperbrothers.com/blog/vue-seo/?utm_source=chatgpt.com "SEO in Vue.js With Vue-Meta, Vue Router, and Other Useful Tools")).
- Provides product information, pricing, and an “Order Now” button that redirects to the billing subdomain.
### 2. FOSSBilling (Payment & Client Area)
- An **Apache 2.0-licensed** PHP/MySQL application for invoices, subscriptions, and client management ([FOSSBilling](https://fossbilling.org/docs?utm_source=chatgpt.com "Introduction - FOSSBilling")).
- Extended with the **community Pterodactyl-FOSSBilling module** to automate server provisioning (orders → API calls) ([GitHub](https://github.com/geron74784/Pterodactyl-FOSSBilling?utm_source=chatgpt.com "geron74784/Pterodactyl-FOSSBilling - GitHub")).
- Runs on a dedicated subdomain (e.g., `billing.example.com`) behind Nginx.
### 3. Pterodactyl Panel & Wings (Game-Server Management)
- **Panel**: A Laravel/React web UI that manages user accounts, allocations, and Docker containers ([Pterodactyl](https://pterodactyl.io/?utm_source=chatgpt.com "Pterodactyl Panel")).    
- **Wings**: The daemon on each host node that pulls images and launches game servers in isolated Docker containers.
- Users access their servers at `panel.example.com` to view console, file manager, and resource usage.
### 4. Nginx Reverse Proxy
- Routes requests to the correct service based on hostname or path (Vue site, FOSSBilling, Panel) ([Gist](https://gist.github.com/TheDevFreak/94b702f4c802fd76e41880ef1da3d9e7?utm_source=chatgpt.com "Pterodactyl Panel Behind an NGINX Reverse Proxy - GitHub Gist")).
- Terminates TLS (Let’s Encrypt) and enforces HSTS, then forwards to backend apps over HTTP.
---
## Workflow
1. **Visit Frontend**  
    Customer lands on `www.example.com` (Vue), reads features/pricing, clicks “Order Now.
2. **Billing Checkout**  
    Redirect to `billing.example.com`; user registers/logs in, selects plan, and pays via Stripe/PayPal ([FOSSBilling](https://fossbilling.org/docs/faq?utm_source=chatgpt.com "FAQ | FOSSBilling")).
3. **Automated Provisioning**  
    FOSSBilling’s Pterodactyl module uses the stored API key to call `POST /api/application/servers` on the Panel, creating a new server instance with defined CPU/RAM/Disk limits.
4. **Server Ready**  
    Upon successful API response, FOSSBilling marks the order active and displays a “Go to Panel” button.
5. **Panel Access**  
    User clicks through to `panel.example.com`, logs in (same credentials), and sees their new Minecraft server console/file manager.
---
## Deployment Steps
### Vue Frontend

```bash
npm run build
cp -r dist/* /var/www/html/vue
```

Configure Nginx with:

````nginx
server {
  listen 80; server_name www.example.com;
  root /var/www/html/vue; index index.html;
  location / { try_files $uri $uri/ /index.html; }
}
``` :contentReference[oaicite:7]{index=7}

### ### FOSSBilling  
1. Clone & install dependencies (`composer install`, `npm install`, `.env` setup) :contentReference[oaicite:8]{index=8}  
2. Clone Pterodactyl-FOSSBilling module into `library/Server/Manager/Pterodactyl` :contentReference[oaicite:9]{index=9}  
3. Migrate database and seed.  
4. Configure Nginx vhost for `billing.example.com`.

### ### Pterodactyl Panel & Wings  
1. Follow official getting-started guide to install Panel (`/var/www/pterodactyl`) and Wings on host nodes :contentReference[oaicite:10]{index=10}.  
2. Build frontend assets with `npm run build:prod` :contentReference[oaicite:11]{index=11}.  
3. Proxy through Nginx at `panel.example.com`.

---

## ## Security & Maintenance

- **TLS everywhere**: Use Certbot to automate Let’s Encrypt certificates for all subdomains.  
- **Firewall**: Only expose ports 80/443; restrict Wings ports to the Panel host.  
- **Backups**: Automate MySQL and Pterodactyl configuration backups.  
- **Updates**: Pull latest FOSSBilling repo and community module; `git pull` upstream Panel updates; rebuild assets.

---

## ## Further Considerations

- **Single Sign-On**: Future work could centralize authentication (e.g., OAuth2 proxy) so users need only one credential for FOSSBilling & Panel.  
- **Metrics & Monitoring**: Add Prometheus + Grafana or similar to track server health and billing metrics.  
- **Scaling**: Add additional Wings nodes behind a load balancer as demand grows.

---

This architecture delivers a **cost-free**, fully open-source stack for **automated Minecraft server hosting**, with a modern Vue frontend, zero-cost FOSSBilling billing, and robust Pterodactyl orchestration.
::contentReference[oaicite:12]{index=12}
````
