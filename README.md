# Hidden Terps Grow Tracker

Site statique (aucun framework) + Supabase (auth, base Postgres, stockage photos) + Vercel.

## 1. Supabase
1. Crée un projet sur supabase.com.
2. SQL Editor > colle et exécute `supabase/schema.sql`.
3. Authentication > Providers > Email : actif. (Pour tester vite, désactive "Confirm email".)
4. Project Settings > API : copie **Project URL** et la clé **anon public**.
5. Authentication > URL Configuration : mets l'URL Vercel dans *Site URL*.

## 2. Vercel
1. Pousse ce dossier sur GitHub, puis Vercel > Add New > Project > importe le dépôt.
2. Variables d'environnement : `SUPABASE_URL`, `SUPABASE_ANON_KEY` (voir `.env.example`).
3. Deploy. (`vercel.json` lance `node build.js`, qui génère `public/config.js`.)

## 3. Usage
- Crée ton compte sur l'écran de connexion, puis Plus > Charger les données de démo si tu veux.
- iPhone : Safari > Partager > Sur l'écran d'accueil. Le scan caméra exige HTTPS (fourni par Vercel).
- Les QR encodent l'URL de production : déploie d'abord, imprime ensuite.

## Structure / ajouter des modules
- `public/index.html` : toute l'app. Chaque écran est une fonction (`home`, `plantsPage`, `sessionPage`…), enregistrée dans `render()`.
- Données : 4 tables (`sessions`, `plants`, `events`, `lots`) avec une colonne `data jsonb`. Un nouveau module (arrosage avancé, calendrier, breeding…) = de nouveaux types d'événements ou une nouvelle table créée par la même boucle SQL.
- `save()` compare l'état et n'envoie que les lignes modifiées ; les photos partent dans le bucket `photos/<user_id>/`.

## v2 — migration depuis la v1
1. Ré-exécute `supabase/schema.sql` (idempotent : renomme `events` en `plant_events`, ajoute les nouvelles tables, les vues, le bucket privé et `trace(uuid)`).
2. Redéploie. Les plantes existantes reçoivent leur UUID automatiquement.
3. **Réimprime les étiquettes** : les QR v2 encodent l'UUID de la plante (`#/p/<uuid>`). Les anciens QR (ID lisible) restent lisibles par un utilisateur connecté, mais plus par le public.
- Identité : `plants.uuid` (unique globalement, jamais modifié) + ID lisible HT-… (unique par utilisateur, jamais dérivé de la session après création).
- Tables : `plants`, `plant_events`, `genetics`, `tasks`, `environmental_readings`, `feeding_recipes`, `pollinations`, `lots`. Vues : `waterings`, `treatments`, `harvests`, `drying`, `curing`, `storage`, `health_issues`, `lot_plants`.
