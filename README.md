# Literature Graph

**Turn any research field into an interactive citation network. Ask it questions, inspect papers, and write a literature review from the papers you select. It runs on your own computer, with no API keys and no cost.**

![Literature Graph overview](lit-graph-overview.png)

Enter a topic. The app builds a citation graph from [OpenAlex](https://openalex.org): each dot is a paper, each line is a citation, and each color is a research cluster. The language model runs locally through [Ollama](https://ollama.com).

Examples from real runs:

| Topic | Papers | Citation links | Clusters |
|---|---|---|---|
| knowledge graph completion | 77 | 724 | 3 |
| drug target prediction | 93 | 467 | 4 |

## Contents

- [What you can do](#what-you-can-do)
- [1. Build a graph](#1-build-a-graph)
- [2. Explore the graph](#2-explore-the-graph)
- [3. Ask the graph](#3-ask-the-graph)
- [4. Inspect any paper](#4-inspect-any-paper)
- [5. Select papers and write a literature review](#5-select-papers-and-write-a-literature-review)
- [How it works](#how-it-works)
- [Local, free and private](#local-free-and-private)
- [Quick start](#quick-start)
- [Evaluation](#evaluation)
- [Limitations](#limitations)
- [Project structure](#project-structure)
- [Roadmap](#roadmap)
- [Part of ResearchDesk](#part-of-researchdesk)
- [Author](#author) · [License](#license)

## What you can do

| Step | What happens |
|---|---|
| **Build** | Type a topic, pick a size (about 60, 100 or 150 papers), get a clustered citation graph |
| **Explore** | Zoom, pan, drag nodes, hide clutter, size nodes by citations or PageRank |
| **Ask** | Plain-English questions answered by graph algorithms: clusters, influential papers, bridges, rising work, timelines, reading order, predicted links |
| **Inspect** | Click a paper for authors, venue, date, citations, DOI, abstract, datasets, metrics and BibTeX |
| **Review** | Select 2 to 15 papers and get a cited literature review with a real reference list |

The interface is a sidebar app with an Ollama status light, a language-model picker and a light/dark switch. The Literature Graph tab has example topic chips, a size menu, a mode menu (Fast or Agent) and a review-style menu (themes or short summary).

## 1. Build a graph

Choose a topic and a size, then click **Build graph**. The status line reports the result, for example: *"knowledge graph completion": 77 papers · 724 citation links · 3 clusters (1 unconnected papers removed).*

How the graph is made:

1. **Seed papers.** The top papers for your topic from OpenAlex's relevance search.
2. **Background papers.** References cited by at least two seed papers are added, since they are the foundations of the field.
3. **Citation links.** A link is drawn whenever one fetched paper cites another. Papers with no links are removed, and the status line says how many.
4. **Analysis.** The app computes PageRank, betweenness centrality and modularity communities, and gives each cluster a short keyword label.
5. **Ids.** Papers are named `P1`, `P2`, ... in order of total citations, so you can refer to them in questions.

## 2. Explore the graph

![Graph overview](lit-graph-overview.png)

- **Scroll** to zoom, **drag empty space** to pan, **drag a node** to move it.
- **Click** a node to open its details. **Shift-click** to add it to the review basket (selected nodes get an orange ring).
- Untick **Show all links** to remove the clutter: only the links of the selected or hovered paper stay visible.
- Switch node size between **citations** and **PageRank**. **Reset view** restores the default zoom.
- The **legend** under the graph lists each cluster with its keyword label and size, for example *1: relational, network, convolutional, attention (35)*.

## 3. Ask the graph

Questions are answered by **graph algorithms (NetworkX), not by the language model**, so rankings and numbers are exact. The model only chooses a tool in Agent mode and writes a short summary. The papers in the answer **light up on the graph**, and every answer shows the tool used and its parameters.

| Question | Tool | What it computes |
|---|---|---|
| What are the main clusters? | `communities` | Modularity communities with their top papers |
| Most influential papers | `top_papers` | PageRank inside the graph, or total citations; optional "core topic papers only" |
| Which papers bridge clusters? | `bridges` | Betweenness centrality and how many clusters each paper touches |
| Which papers should be connected? | `predict_links` | Adamic-Adar link prediction for pairs that share neighbours but do not cite each other |
| Rising recent papers | `emerging` | Recent papers with the highest citations per year |
| Timeline of the clusters | `timeline` | Papers per year in each large cluster |
| Where should I start reading? | `reading_list` | Foundational papers first (older, high PageRank), then recent rising ones |
| Path between P1 and P5 | `find_path` | Shortest citation connection between two papers |
| Who cites P3? | `neighbors` | What a paper cites and what cites it |

**Two modes:** *Fast* matches the question with rules and asks the model only if no rule fits. *Agent* always lets the model choose the tool. Each answer shows which one picked it.

### Example: timeline of the clusters

![Timeline of the clusters](lit-graph-timeline.png)

For *knowledge graph completion*, "Timeline of the clusters" ran the `timeline` tool and drew papers per year for each large cluster, from 2011 to 2025: cluster 1 (*relational, network, convolutional, attention*, 35 papers), cluster 2 (*embedding, learning, models, relation*, 31 papers) and cluster 3 (*temporal, tucker, tensor, factorization*, 11 papers). The bars make it easy to see when each line of work grew.

### Example: where to start reading

![Where should I start reading](lit-graph-reading.png)

For the same graph, the ranking of influential papers (PageRank inside the graph, core topic papers only) put these on top:

1. *Learning Entity and Relation Embeddings for Knowledge Graph Completion* (2015, 3,850 citations)
2. *A Re-evaluation of Knowledge Graph Completion Methods* (2020)
3. *KG-BERT: BERT for Knowledge Graph Completion* (2019)
4. *TuckER: Tensor Factorization for Knowledge Graph Completion* (2019)
5. *Temporal Knowledge Graph Completion: A Survey* (2023)

**An honest note:** in this run, Agent mode chose `top_papers` for "Where should I start reading?" instead of the `reading_list` tool. The result is still useful, but it shows that a 3B model can pick a neighbouring tool. Fast mode routes this question to `reading_list` by rule. The evaluation below measures this.

## 4. Inspect any paper

![Paper details](lit-graph-details.png)

Click a node to open the **Paper details** box. The data comes live from OpenAlex: title, authors, venue, publication date, citation count, DOI, cluster, abstract, BibTeX, and a PDF link when one is open access. The papers connected to the selected one are highlighted on the graph.

**Datasets, metrics and reported figures** are extracted from the abstract in two ways:

- **Pattern-based (instant).** Known dataset names (for example FB15k-237, WN18RR, DrugBank, KIBA), metric names (accuracy, AUC, F1, MRR, Hits@k and others), and abstract sentences that pair a metric with a number. The supporting sentence is always shown.
- **Extract with AI (button).** The local model returns method, datasets and metrics and must give a **word-for-word quote** for each. The app checks that the quote really appears in the abstract and marks each item **verified** or **unverified**. Ignore the unverified ones.

**When there is nothing to extract, the box says so.** The screenshot shows *Support-Vector Networks* (Cortes and Vapnik, 1995, 62,322 citations), where OpenAlex has no abstract, so the panel reports that nothing can be extracted. Other papers, such as *Adam*, have an abstract that names no dataset or score, and the panel reports "none named in the abstract". The tool prefers an honest empty result to a guess. Per-dataset scores and tables live in the full text, which this panel does not read.

## 5. Select papers and write a literature review

Build a **review basket** of 2 to 15 papers in any of these ways:

- **Shift-click** nodes on the graph.
- Click **Add to review** in the details box.
- Ask a question, then click **Add highlighted** to take the papers it lit up.
- Pick a cluster and click **Add top 6**.

Click **Write review** and choose a themed review (about 300 words) or a short summary (about 120 words).

- The model writes from the selected papers' abstracts, citing `[1]`, `[2]`.
- The **reference list is built from real OpenAlex data**, not written by the model, so it cannot contain invented papers.
- Citation numbers that do not exist are **flagged**, papers never cited are listed, and papers with no abstract get a warning.
- Click a citation to jump to its reference. Copy the text, references or BibTeX, or download everything as Markdown.

<!-- Add a screenshot of a finished review, then remove these comment markers:
![Literature review](lit-graph-review.png)
-->

## How it works

```
topic --> OpenAlex search --> seed papers + frequently cited references
      --> citation links --> NetworkX graph
      --> PageRank, betweenness, communities, labels

question --> rules (Fast) or LLM tool choice (Agent) --> graph algorithm --> exact result
         --> highlighted on the graph + short summary

click a paper --> OpenAlex record --> abstract --> pattern extraction (+ optional AI with verified quotes)

selected papers --> abstracts --> LLM writes the review --> Python builds the references
                --> checks for invented citation numbers
```

**Design rule: the model writes prose, and Python supplies the facts.** Titles, authors, links, rankings, counts and reference lists never come from the language model.

## Local, free and private

- **No cost.** An open model run by Ollama. No API key, billing or usage limit.
- **Text generation stays on your machine.** Summaries, extraction and reviews are written by a model running on your computer.
- **Real data.** Papers, citations and references come from OpenAlex.
- **Transparent.** Every answer shows the tool used and its parameters.

**What is sent over the internet:** your topic and the ids of papers you inspect or select go to OpenAlex. Model downloads come from Ollama once at setup. The optional `SCHOLAR_EMAIL` is sent to OpenAlex as a contact address. Nothing else leaves your computer.

**What it does not do:** it is not fully offline, and there is **no login**. The server listens on `127.0.0.1` only, so never expose it to the internet or port-forward it. It is not an audited security product.

## Quick start

**Requirements:** Python 3.9+ and [Ollama](https://ollama.com/download).

```bash
git clone https://github.com/hajigul/ResearchDesk.git
cd ResearchDesk

python3 -m venv agent-env
source agent-env/bin/activate        # Windows: agent-env\Scripts\activate
pip install -r requirements.txt

ollama pull llama3.2:3b              # language model (about 2 GB)
python pro_server.py
```

Keep the Ollama app running. The app opens at http://127.0.0.1:8003. Open **Literature Graph** in the sidebar, enter a topic and click **Build graph**.

Optional: `export SCHOLAR_EMAIL="you@example.com"` before starting, since OpenAlex serves identified clients more reliably. If a pip download times out, use `pip install --resume-retries 5 --default-timeout 100 -r requirements.txt`. A larger model such as `llama3.1:8b` (set `SCHOLAR_MODEL`) usually writes better reviews but is slower.

## Evaluation

```bash
python eval_graph.py "knowledge graph completion"
```

It reports:

- **Tool-choice accuracy** on 12 questions, in Fast mode (rules first) and Agent mode (model only). The numbers in answers are exact by construction, since NetworkX computes them, so the open question is whether the right tool is chosen.
- **Link-prediction AUC.** It hides 10% of the citation links, then measures how often Adamic-Adar scores a hidden link above a random non-link (0.5 is chance).

| Topic | Papers | Links | Tool choice (Fast) | Tool choice (Agent) | Link-prediction AUC |
|---|---|---|---|---|---|
| _fill in after running_ | | | | | |

**Observed so far:** in a manual run, Agent mode picked `top_papers` for "Where should I start reading?" instead of `reading_list`, while the other questions shown above used the expected tools.

**Not evaluated yet:** the paper-details extraction and the literature review. A fair test for extraction is to read 20 abstracts, record the datasets and numbers they state, and count hits, misses and inventions. For reviews, ask a colleague to mark unsupported sentences.

## Limitations

- **The graph is only as good as OpenAlex's data.** It contains seed papers plus frequently cited references. Very general, highly cited papers can enter as background (for example *Adam*, or classics such as *Support-Vector Networks* in a drug-target graph). Use "core topic papers only" in the influence ranking to exclude them.
- **Some papers have no abstract in OpenAlex.** Details, extraction and reviews are weaker for them, and the app says so.
- **Duplicate versions.** The same paper can appear twice when OpenAlex lists a preprint and a published version separately. Rankings treat them as different papers.
- **PageRank and clusters are computed inside this small sample**, not across the whole literature.
- **Cluster labels are frequent title words**, so they can be generic.
- **Predicted links and reading lists are heuristics**, meant as reading suggestions, not facts. "Rising" favors papers a few years old over brand-new ones.
- **Bridge rankings may use sampled betweenness** on larger graphs, so they are approximate.
- **Agent mode can pick the wrong tool.** A 3B model sometimes chooses a neighbouring tool. Use Fast mode for the supported questions.
- **Dataset and metric extraction uses abstracts only.** Many abstracts name no dataset or score, the pattern matcher can produce false names, and the AI button can misread an abstract even when its quote is real.
- **Reviews use abstracts only** (about 550 characters each) and are written by a small local model, so they can be superficial or overstate a claim. Verify every statement against the papers.
- **One graph per server.** The server holds a single graph. Building a new one replaces it, so do not build in two browser tabs at once, and rebuild before asking if the header does not match your topic.
- **Caching.** OpenAlex responses are cached on disk for 6 hours, so rebuilding the same topic is fast but may not show brand-new papers.
- **Limits:** 2 to 15 papers per review, and one task at a time to avoid overloading a laptop.

## Project structure

```
ResearchDesk/
├── pro_server.py      # entry point: ResearchDesk plus the Literature Graph tab
├── ext_00_core.js     # helper that adds a tab to the sidebar
├── ext_graph.js       # Literature Graph front end: canvas, details, basket, review
├── litgraph.py        # graph building and algorithms (NetworkX)
├── graph_agent.py     # maps a question to a graph tool and streams the answer
├── graph_tools.py     # paper details, abstract extraction, literature review
├── eval_graph.py      # evaluation of the graph agent
├── suite_server.py, suite.html            # ResearchDesk base app
├── tools.py, config.py                    # OpenAlex/arXiv access, settings
├── agent.py, related_work.py, citecheck.py, thesis_qa.py   # the other four tools
└── *.png              # screenshots
```

## Roadmap

- Fetch abstracts for every graph paper, and add papers that cite the seed papers
- Cluster names written by the local model
- Save, load and export graphs (GraphML for Gephi)
- Check that questions run against the graph the page is showing
- Literature Matrix Extractor: datasets, methods and results from full-text PDFs, each value tied to a page and a verified quote
- Evaluation of extraction and reviews with hand-labeled data

## Part of ResearchDesk

The Literature Graph is one tab of **ResearchDesk**, a private research assistant that also includes paper **Discover**, a **Related Work** drafter, a **Citation Check** for reference lists, and **Thesis Q&A** over your own PDFs. They share the same rule: the model writes prose, and real data supplies the facts. Thesis Q&A needs `ollama pull nomic-embed-text` and a `thesis/` folder with your PDFs, which should stay out of Git.

## Author

**Haji Gul**, PhD candidate (AI), Universiti Brunei Darussalam.
Research interests: knowledge graph completion, link prediction, LLM-augmented graph reasoning.

GitHub: [@hajigul](https://github.com/hajigul)

## License

MIT License. See `LICENSE`.

## Acknowledgments

[OpenAlex](https://openalex.org), [Ollama](https://ollama.com), [NetworkX](https://networkx.org).
