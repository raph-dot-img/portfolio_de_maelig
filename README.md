# Portfolio de Maëlig

Code source du CV en ligne de **Maëlig Chaussy**, publié sur [maelig.eu](https://maelig.eu).

Site statique (HTML et CSS, sans JavaScript ni dépendance) servi par nginx dans Docker, et exposé sur Internet par un tunnel [FRP](https://github.com/fatedier/frp).

## Déploiement

```sh
git clone https://github.com/raph-dot-img/portfolio_de_maelig.git
cd portfolio_de_maelig
cp .env.example .env   # puis remplir les valeurs FRP
docker compose up -d
```

- Le site répond en local sur `http://127.0.0.1:45889`.
- `frpc` relaie le trafic en TCP vers le serveur FRP. Le HTTPS est géré en amont par Nginx Reverse Proxy, qui doit pointer le domaine (et ses sous-domaines) vers `FRP_SERVER_REMOTE_PORT` et transmettre l'en-tête `X-Forwarded-For`.
- Le journal d'accès est conservé dans `logs/acces.log`.

Après une modification d'un fichier du site : `docker compose restart portfolio` (les fichiers sont montés un par un, et beaucoup d'éditeurs remplacent le fichier au lieu de le modifier, ce que le conteneur ne voit pas).

## Développement

```sh
python3 serve.py        # http://localhost:8080
```

Serveur de développement uniquement : il reproduit les pages d'erreur et la page 418, mais n'est pas fait pour la production.

## Licence

- **Le code est sous licence [MIT](LICENSE)** : vous pouvez le réutiliser, le modifier et le redistribuer librement, en gardant la mention de copyright.
- **Le contenu personnel de Maëlig en est exclu** et reste tous droits réservés : textes du CV, politique de confidentialité, nom, coordonnées. Ce sont des données personnelles, et Maëlig est mineur·e : merci de ne pas les réutiliser. Si vous reprenez le code, remplacez tout ce contenu par le vôtre. Le détail est dans [NOTICE](NOTICE).

Un problème sur le site ? Ouvrez une [issue](https://github.com/raph-dot-img/portfolio_de_maelig/issues/).
