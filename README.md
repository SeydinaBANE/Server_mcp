# 🤖 Server MCP - Agent LangGraph avec outils Math & Météo

> 🎯 **Objectif pédagogique** : Ce projet a pour but de montrer comment utiliser le protocole **MCP (Model Context Protocol)** pour connecter un agent IA à des outils externes, en utilisant les deux types de transport  : `stdio` et `streamable_http`.

Un projet d'agent IA basé sur **LangChain**, **LangGraph** et le protocole **MCP (Model Context Protocol)**, connectant un modèle LLM Groq à des outils externes via des serveurs MCP, avec observabilité via **LangSmith** et **Langfuse**.

### Qu'est-ce que MCP ?

Le **Model Context Protocol** est un standard ouvert qui permet à un LLM de communiquer avec des outils externes (serveurs MCP) de manière structurée. Il définit comment un agent peut découvrir, appeler et recevoir les résultats d'outils distants, quel que soit le langage ou la plateforme utilisée.

Ce projet illustre deux modes de communication MCP :

| Transport | Cas d'usage | Exemple dans ce projet |
|---|---|---|
| `stdio` | Outil local, lancé par le client | `mathserver.py` |
| `streamable_http` | Outil distant, serveur indépendant | `weather.py` |

---

## 📁 Structure du projet

```
Server_mcp/
├── src/
│   ├── client.py          # Agent principal (LangGraph + Groq)
│   ├── mathserver.py      # Serveur MCP — outils mathématiques (stdio)
│   └── weather.py         # Serveur MCP — outil météo (streamable_http)
├── .env                   # Clés API (non versionné)
├── .env.example           # Modèle de variables d'environnement
├── .gitignore
├── .python-version        
├── .venv/                
├── pyproject.toml         # Configuration du projet
├── requitements.txt      
├── uv.lock               
└── README.md
```

---

## ⚙️ Prérequis

- Python 3.11+ (voir `.python-version`)
- [uv](https://docs.astral.sh/uv/) — gestionnaire de paquets (recommandé)
- Un compte [Groq](https://console.groq.com) pour la clé API LLM
- Un compte [LangSmith](https://smith.langchain.com) pour le tracing
- Un compte [Langfuse](https://cloud.langfuse.com) pour l'observabilité

---

## 🚀 Installation

### Avec `uv` (recommandé)

```bash
cd Server_mcp
uv sync
```

### Avec `pip` classique

```bash
cd Server_mcp
python -m venv .venv
source .venv/bin/activate  # Sur Windows : .venv\Scripts\activate
pip install -r requitements.txt
```

### Configurer les variables d'environnement

Copier le fichier `.env.example` et remplir les valeurs :

```bash
cp .env.example .env
```

Contenu du `.env` :

```env
# LLM
GROQ_API_KEY=votre_clé_groq_ici

# LangSmith (tracing & évaluation)
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=votre_clé_langsmith_ici
LANGCHAIN_PROJECT=server_mcp

# Langfuse (observabilité & analytics)
LANGFUSE_PUBLIC_KEY=votre_clé_publique_langfuse_ici
LANGFUSE_SECRET_KEY=votre_clé_secrète_langfuse_ici
LANGFUSE_HOST=https://cloud.langfuse.com
```



---

## 🧩 Architecture

Ce projet utilise deux types de transport MCP :

| Fichier | Transport | Lancement |
|---|---|---|
| `src/mathserver.py` | `stdio` | Automatique via le client |
| `src/weather.py` | `streamable_http` | Manuel sur le port 8000 |

```
src/client.py
    ├── MultiServerMCPClient
    │       ├── math (stdio)    → src/mathserver.py
    │       └── weather (http)  → src/weather.py :8000
    ├── ChatGroq (openai/gpt-oss-120b)
    ├── create_agent (LangChain)
    ├── LangSmith  ──────────────────────→ smith.langchain.com
    └── Langfuse   ──────────────────────→ cloud.langfuse.com
```

---

## 📊 Observabilité

### LangSmith
Trace automatiquement toutes les invocations de l'agent, les appels aux outils MCP et les réponses du modèle. Accessible sur [smith.langchain.com](https://smith.langchain.com) sous le projet `server_mcp`.

### Langfuse
Fournit des analytics détaillés sur les coûts, latences et qualité des réponses LLM. Accessible sur [cloud.langfuse.com](https://cloud.langfuse.com).

---

## 🛠️ Les outils disponibles

### Serveur Math (`src/mathserver.py`)
| Outil | Description | Paramètres |
|---|---|---|
| `add` | Additionne deux nombres | `a: float, b: float` |
| `subtract` | Soustrait deux nombres | `a: float, b: float` |

### Serveur Météo (`src/weather.py`)
| Outil | Description | Paramètres |
|---|---|---|
| `get_weather` | Retourne la météo d'une ville | `location: str` |

---

## ▶️ Lancer le projet

### Étape 1 — Démarrer le serveur météo (Terminal 1)

```bash
source .venv/bin/activate
python src/weather.py
```

Le serveur démarre sur `http://127.0.0.1:8000/mcp`

### Étape 2 — Lancer le client agent (Terminal 2)

```bash
source .venv/bin/activate
python src/client.py
```

### Résultat attendu

```
Les outils disponibles: ['add', 'subtract', 'get_weather']
La réponse à votre question: (4+5) = 9 et (54-50) = 4
```

---

## 🐛 Erreurs fréquentes

| Erreur | Cause | Solution |
|---|---|---|
| `Missing 'transport' key` | Clé `transport` absente dans la config | Ajouter `"transport": "stdio"` ou `"streamable_http"` |
| `SyntaxError: forgot a comma` | `print("texte" variable)` | Ajouter une virgule : `print("texte", variable)` |
| `object dict can't be used in await` | Utilisation de `.invoke()` au lieu de `.ainvoke()` | Remplacer par `await agent.ainvoke(...)` |
| `streamable-http` non reconnu | Tiret au lieu d'underscore | Utiliser `streamable_http` |
| Push GitHub bloqué | Fichier `.env` commité avec des clés | Révoquer les clés, retirer `.env` de l'historique Git |

---

## 📚 Technologies utilisées

- [LangChain](https://python.langchain.com/) - Framework LLM
- [LangGraph](https://langchain-ai.github.io/langgraph/) - Agent ReAct
- [langchain-mcp-adapters](https://github.com/langchain-ai/langchain-mcp-adapters) -Intégration MCP
- [FastMCP](https://github.com/jlowin/fastmcp) - Création de serveurs MCP
- [Groq](https://groq.com/) — Inférence LLM ultra-rapide
- [LangSmith](https://smith.langchain.com/) - Tracing & évaluation des chaînes LLM
- [Langfuse](https://langfuse.com/) - Observabilité & analytics LLM
- [uv](https://docs.astral.sh/uv/) - Gestionnaire de paquets moderne
- [python-dotenv](https://pypi.org/project/python-dotenv/)- Gestion des variables d'environnement

---

## 👤 Auteur

**BANE Seydina Mouhamet**
