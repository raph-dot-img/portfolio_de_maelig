# CLAUDE.md

Notes pour les futures sessions de Claude Code sur ce dépôt.

## Le projet

CV en ligne de Maëlig Chaussy (élève en Seconde Bac Pro CIEL), publié sur https://maelig.eu. Tout le site est en français.
L'infrastructure reprend celle du portfolio de Jean-Baptiste (`raph-dot-img/portfolio_de_jb`), sans son contenu.

- Site statique : `index.html`, `confidentialite.html`, `style.css`. Aucun framework, aucun build, pas de JavaScript pour l'instant.
- Couleurs en variables CSS dans `:root` (sarcelle `#0f6e6e`), avec un thème sombre via `prefers-color-scheme`. Polices système uniquement.
- Noms de classes et commentaires en français. Le site vouvoie le visiteur.
- Contacts : `privacy@canevet.eu` (RGPD), `webmaster@canevet.eu` (serveur, journaux), issues GitHub pour les problèmes du site.
- Pas de téléphone ni d'adresse postale sur le site (Maëlig est mineur·e) : seulement l'adresse mail dédiée.
- Le texte de `confidentialite.html` est fourni par le propriétaire du dépôt : n'en changer le fond qu'à sa demande.

## Contraintes à respecter

- **Aucune ressource externe** (police, CDN, analytics...) : la CSP de `nginx/securite.conf` n'autorise que `'self'`. Pas de style ou de script inline (attributs `style=""` compris) : ils seraient bloqués.
- Ne jamais versionner `.env` (seul `.env.example` l'est).

## Infrastructure

- `docker-compose.yml` : `portfolio` (nginx:alpine, port local 45889) + `frpc` (tunnel TCP vers le serveur FRP). Le HTTPS est terminé par Nginx Reverse Proxy sur le serveur distant (VPS Kyun.sh). Serveur et relais sont gérés par Raphaël, comme pour le site de JB.
- Le proxy FRP s'appelle `portfolio_maelig` et le port distant est distinct de celui de JB : les deux sites peuvent partager le même frps.
- La vraie IP du visiteur vient de `X-Forwarded-For` (module realip, adresses privées de confiance).
- Les fichiers du site sont montés un par un. **Toute nouvelle page publique** doit être ajoutée dans les volumes de `docker-compose.yml` ET dans `RACINES_PUBLIQUES` de `serve.py`, et si besoin dans `sitemap.xml`.
- Limite de débit par IP (`limit_req`, 10 req/s, rafale de 40 → page 429) et corps de requête limité à 1 Ko.
- Le journal d'accès (`logs/acces.log`) contient les IP des visiteurs : à garder en tête pour la politique de confidentialité.

## Pages d'erreur et piège à scanners

- `erreurs/*.html` : pages d'erreur personnalisées, servies uniquement en interne par nginx.
- Les chemins sensibles (`/.env`, `/admin`, `/wp-login.php`...) renvoient un **418** avec l'IP du client, insérée par SSI (`<!--# echo var="remote_addr" -->`).
- La liste des chemins existe en double : regex dans `nginx/default.conf` et `CHEMINS_SCANNERS` dans `serve.py`. Les garder identiques.

## Tester

- Dev : `python3 serve.py [port]` (reproduit erreurs et 418 avec IP).
- Prod locale : `cp .env.example .env && docker compose up -d portfolio`, puis http://127.0.0.1:45889. Redémarrer le conteneur après chaque modification.
- Vérifier la config nginx : `docker run --rm -v $PWD/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro -v $PWD/nginx/securite.conf:/etc/nginx/securite.conf:ro nginx:alpine nginx -t`.
- Vérifier le responsive jusqu'à 320 px de large, sans défilement horizontal.

## Licence

Code sous Microsoft Reference Source License, Ms-RSL (`LICENSE`).
