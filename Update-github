// netlify/functions/update-github.js
// Cette fonction tourne côté serveur Netlify.
// Elle lit GH_TOKEN depuis les variables d’environnement (jamais exposé au navigateur).

const https = require(“https”);

exports.handler = async function (event) {
// Autoriser uniquement les requêtes POST
if (event.httpMethod !== “POST”) {
return { statusCode: 405, body: JSON.stringify({ error: “Méthode non autorisée” }) };
}

// Récupérer le token depuis l’environnement Netlify (sécurisé)
const GH_TOKEN = process.env.GH_TOKEN;
if (!GH_TOKEN) {
return {
statusCode: 500,
body: JSON.stringify({ error: “GH_TOKEN non configuré dans les variables d’environnement Netlify.” })
};
}

let payload;
try {
payload = JSON.parse(event.body);
} catch (e) {
return { statusCode: 400, body: JSON.stringify({ error: “Corps de requête JSON invalide.” }) };
}

const { repo, file, branch, htmlContent, commitMessage } = payload;

if (!repo || !file || !htmlContent) {
return {
statusCode: 400,
body: JSON.stringify({ error: “Champs manquants : repo, file, htmlContent requis.” })
};
}

const targetBranch = branch || “main”;
const targetFile   = file || “index.html”;
const commitMsg    = commitMessage || `Mise à jour site - ${new Date().toLocaleString("fr-FR")}`;

// ── Étape 1 : récupérer le SHA actuel du fichier ──────────────────────────
let sha;
try {
sha = await getFileSHA(GH_TOKEN, repo, targetFile, targetBranch);
} catch (e) {
return {
statusCode: 502,
body: JSON.stringify({ error: “Impossible de lire le fichier sur GitHub : “ + e.message })
};
}

// ── Étape 2 : encoder le HTML en base64 ───────────────────────────────────
const base64Content = Buffer.from(htmlContent, “utf8”).toString(“base64”);

// ── Étape 3 : envoyer la mise à jour à GitHub ─────────────────────────────
try {
await pushToGitHub(GH_TOKEN, repo, targetFile, targetBranch, base64Content, sha, commitMsg);
} catch (e) {
return {
statusCode: 502,
body: JSON.stringify({ error: “Échec du push GitHub : “ + e.message })
};
}

return {
statusCode: 200,
headers: { “Content-Type”: “application/json”, “Access-Control-Allow-Origin”: “*” },
body: JSON.stringify({ success: true, message: “Fichier mis à jour sur GitHub avec succès !” })
};
};

// ── Helpers ──────────────────────────────────────────────────────────────────

function githubRequest(method, path, token, body) {
return new Promise((resolve, reject) => {
const options = {
hostname: “api.github.com”,
path: path,
method: method,
headers: {
“Authorization”: `token ${token}`,
“Accept”: “application/vnd.github.v3+json”,
“User-Agent”: “GalerieAfrik-Netlify-Function”,
“Content-Type”: “application/json”
}
};

```
const req = https.request(options, (res) => {
  let data = "";
  res.on("data", chunk => data += chunk);
  res.on("end", () => {
    try {
      const parsed = JSON.parse(data);
      if (res.statusCode >= 400) {
        reject(new Error(`GitHub API ${res.statusCode}: ${parsed.message || data}`));
      } else {
        resolve(parsed);
      }
    } catch (e) {
      reject(new Error("Réponse GitHub non-JSON : " + data));
    }
  });
});

req.on("error", reject);
if (body) req.write(JSON.stringify(body));
req.end();
```

});
}

async function getFileSHA(token, repo, file, branch) {
const data = await githubRequest(“GET”, `/repos/${repo}/contents/${file}?ref=${branch}`, token);
return data.sha;
}

async function pushToGitHub(token, repo, file, branch, base64Content, sha, message) {
await githubRequest(“PUT”, `/repos/${repo}/contents/${file}`, token, {
message,
content: base64Content,
sha,
branch
});
}
