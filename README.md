# MM Store Connexion

Extension VS Code pour lancer et superviser `shopify theme dev` directement depuis la barre latérale, sans jamais toucher au terminal.

---

## Ce que fait l'extension

Quand tu développes un thème Shopify, tu dois normalement ouvrir un terminal et taper :

```
shopify theme dev -s ton-store.myshopify.com
```

Cette extension fait exactement ça pour toi — en un clic — et affiche les 3 liens générés directement dans la sidebar de VS Code. L'URL de ta boutique est mémorisée par projet, donc tu n'as jamais à la retaper.

---

## Affichage dans la sidebar

Le panneau **MM Store Connexion** apparaît dans l'Explorer de VS Code (même endroit que MM Snippet X Ray). Il affiche :

```
MM Store Connexion
  ✏  Boutique: ton-store.myshopify.com   ← modifier
  🟢 Statut: Connecté
  🔗 Liens
       ↗ Local
       ↗ Share (aperçu public)
       ↗ Admin (éditeur de thème)
  📄 Voir les logs
```

- **Boutique** — clic pour changer l'URL. Elle est sauvegardée par workspace (chaque projet peut avoir sa propre boutique).
- **Statut** — vert = connecté, rouge = déconnecté.
- **Liens** — apparaissent dès que Shopify CLI les génère. Clic = ouverture dans le navigateur.
- **Voir les logs** — ouvre le panneau Output avec tout ce que Shopify CLI a sorti. Devient `⚠ Logs (erreur)` si une erreur est détectée.

---

## Boutons dans l'en-tête du panneau

| Bouton | Action |
|--------|--------|
| ▶ Play | Lance `shopify theme dev` |
| ⏹ Stop | Arrête le serveur |

---

## Comment ça marche techniquement

### 1. Sauvegarde de la boutique
L'URL est stockée via `context.workspaceState` — spécifique au dossier de travail ouvert dans VS Code. Si tu ouvres un autre projet, tu peux configurer une boutique différente.

### 2. Lancement du serveur
Au clic sur ▶ :
1. Le port 9292 est libéré si un ancien process l'occupe encore (via `netstat` + `process.kill()` en Node.js pur)
2. `shopify theme dev --no-color -s [boutique]` est lancé via `spawn` en arrière-plan
3. Le stdout et stderr sont capturés en temps réel

### 3. Parsing des URLs
L'output de Shopify CLI est analysé avec des regex pour détecter les 3 liens dès qu'ils apparaissent :

```
http://127.0.0.1:9292          → Local
...?preview_theme_id=...        → Share
.../admin/themes/.../editor     → Admin
```

La sidebar se met à jour automatiquement dès qu'un lien est trouvé.

### 4. Authentification automatique
Si Shopify CLI demande une connexion (premier lancement), le lien `accounts.shopify.com` est détecté et ouvert automatiquement dans le navigateur. Un message s'affiche : *"connecte-toi puis reclique sur ▶"*.

### 5. Logs & erreurs
Tout l'output de la CLI est redirigé vers le panneau **Output > MM Store Connexion**. Si une erreur critique est détectée (`Error`, `ETIMEDOUT`, etc.), l'item logs passe en mode `⚠`.

---

## Installation

### Depuis le marketplace VS Code
Cherche **MM Store Connexion** dans l'onglet Extensions de VS Code.

### Depuis un fichier .vsix
`Ctrl+Shift+P` → **Extensions: Install from VSIX** → sélectionne le fichier `.vsix`

---

## Prérequis

- [Shopify CLI](https://shopify.dev/docs/themes/tools/cli) installé et accessible dans le PATH (`shopify` doit fonctionner dans ton terminal)
- Être authentifié au moins une fois via `shopify auth login`
- Windows (le kill de port utilise `netstat` Windows)

---

## Publier une nouvelle version

```bash
# 1. Modifier la version dans package.json
# 2. Packager
npx @vscode/vsce package --no-dependencies

# 3. Uploader sur le marketplace
# marketplace.visualstudio.com → Moon Moon → ... → Update → upload le .vsix
```

---

## Structure du projet

```
co-store/
├── src/
│   └── extension.ts     ← code principal (TreeView + spawn + parsing)
├── media/
│   └── icon.svg         ← icône de la sidebar
├── out/
│   └── extension.js     ← compilé par tsc (ne pas éditer)
├── package.json         ← contributions VS Code (view, commands, menus)
├── tsconfig.json
└── README.md
```

---

## Publisher

**Moon Moon** — [marketplace.visualstudio.com/manage/publishers/moon-moon](https://marketplace.visualstudio.com/manage/publishers/moon-moon)
