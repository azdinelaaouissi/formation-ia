# Formation IA complète — Machine Learning, Deep Learning & IA générative

**Par Azdine Laaouissi** · Site : https://formation-ia-eta.vercel.app

Formation complète en français, en accès libre : cours théoriques suivis de travaux dirigés pratiques (code exécutable et commenté), du premier concept de machine learning jusqu'aux agents IA déployés en production.

## Contenu

| Module | Cours | TD | Dossier |
|---|---|---|---|
| Machine Learning | 12 | 11 (8 supervisé + 3 non supervisé) | `machine_learning/` |
| Deep Learning (PyTorch) | 12 | 5 | `deep_learning/` |
| LLM | 5 | 2 | `ia_generative/llm/` |
| LangChain | 8 | 4 | `ia_generative/langchain/` |
| LangGraph | 6 | 3 | `ia_generative/langgraph/` |
| RAG | 7 | 4 | `ia_generative/rag/` |
| Agents IA | 10 | 5 | `ia_generative/agents/` |
| **Total** | **60** | **34** | |

Ressources complémentaires à la racine : plan de formation RAG (`00_PLAN_FORMATION.html`), plan ML (`machine_learning/00_PLAN_ML.html`), projet pratique RAG commenté ligne par ligne (`projet_pratique_rag.html`), cours complets monolithiques (`cours_rag*.html`, `cours_langchain.html`, `cours_langgraph.html`, `cours_langsmith.html`) et documentation technique FHIR / RAG (`DOCUMENTATION_*.md`).

## Chiffres

- **103 pages HTML** autonomes (60 cours, 34 TD, 9 ressources)
- **7 modules** progressifs : ML → DL → LLM → LangChain → LangGraph → RAG → Agents
- **100 % en français**, thème sombre, diagrammes SVG

## Page d'accueil

`index.html` regroupe tout le contenu : parcours recommandé, recherche instantanée (touche `/`), filtres cours / TD / ressources, et suivi de progression par module (enregistré dans le navigateur, aucune donnée envoyée).

## Approche pédagogique

Chaque TD suit une structure en 9 parties :
1. Récapitulatif (définition, analogie, avantages / inconvénients)
2. Principe de fonctionnement (étapes, schémas SVG, formules)
3. Les données (types, préparation)
4. Les paramètres (impact détaillé avec code + sortie)
5. Les métriques (formules, calcul des erreurs)
6. Implémentation complète (pipeline de A à Z)
7. Expérimentation (preuves concrètes)
8. Erreurs courantes (6 pièges)
9. Résumé (tableaux récapitulatifs)

## Technologies

- Python, scikit-learn, PyTorch
- LangChain v0.3+, LangGraph, LangSmith
- OpenAI API, FAISS, Chroma
- RAGAS (évaluation RAG)
- HTML / CSS statique, sans dépendance à installer

## Utilisation

- **En ligne :** https://formation-ia-eta.vercel.app
- **En local :** cloner le dépôt et ouvrir `index.html` dans un navigateur (ou `python3 -m http.server 8080`).

## Déploiement

Site statique déployé sur Vercel ; chaque `git push` sur `main` redéploie automatiquement. `vercel.json` active les URL propres (`/cours_rag` au lieu de `/cours_rag.html`).

## Auteur

**Azdine Laaouissi** — 2025-2026 · https://github.com/azdinelaaouissi
