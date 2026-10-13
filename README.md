# Medication Interaction Assistant

Interactions between drugs can be a serious issue for medical patients and doctors alike. This is an application that runs locally on your computer and can identify possible interactions that drugs may have for a patient. 

A Dockerized browser application for **informational** medication-interaction screening using a local DDInter-derived knowledge base, plus a CPU-hosted chatbot powered by Ollama. This is a prototype, not a clinically validated medical device or a substitute for a pharmacist/prescriber.

Medication interaction Assitant 

![Application interface](/images/interface.png)

## V3 changes

- Includes the local Ollama chatbot (`qwen2.5:3b` by default), with no paid AI API.
- Supports follow-up questions by passing a short, bounded recent chat history to the local model.
- `/health` reports database status and whether the Ollama service/model appears ready.
- Keeps interaction lookup local. RxNav/RxNorm may still be queried over the internet for best-effort medication-name matching when no local alias matches.
- Persists Ollama model files in a named Docker volume and mounts the local knowledge-base folder read-only.

## Requirements

- Docker Desktop (Windows/macOS) or Docker Engine + Compose plugin (Linux).
- Internet access on first start to pull the Ollama image/model and, optionally, to resolve unmatched medication names through RxNav.
- As a practical starting point, allow about 4–6 GB of available RAM for the 3B quantized model, plus OS and container overhead. CPU speed varies widely.
- The knowledge-base JSON file described below. It is bundled in this source ZIP for demonstration purposes. This will be removed in a future release.

## Install and run

1. Extract this ZIP.
   
   **SKIP STEPS 2 and 3 for now. The medication info is bundled for demonstration**

~~2. Create a `data` folder beside `compose.yaml` if it does not exist.~~

~~3. Obtain the compatible JSON knowledge base from the upstream project: <https://github.com/pbiondich/openmrs-ddi-knowledge-base>. Place it at:~~

   ~~`data/ddi_knowledge_base.json`~~

   ~~The app expects the schema used by that project: top-level `drugs`, `interactions`, and `mechanisms` fields. Confirm the downloaded file matches this schema before use.~~

4. From the project directory, run:

   ```bash
   docker compose up --build -d
   ```

5. On first start, Docker downloads Ollama and `qwen2.5:3b`; allow several minutes. Open <http://localhost:8000> when the app is running.
6. Check health at <http://localhost:8000/health>. `chatbot.status` should become `ready` after the model download finishes.

Useful commands:

```bash
docker compose ps
docker compose logs -f ollama-model-init
docker compose logs -f ollama
docker compose logs -f medication-checker
docker compose down
```

To change the model, update **both** `OLLAMA_MODEL` in `compose.yaml` and the model tag in the `ollama-model-init` command so they match. A smaller model may use less memory but may give less capable explanations.

## Troubleshooting

- If screening says the database is missing, check `data/ddi_knowledge_base.json` and inspect the logs.
- If the chatbot is unavailable, run `docker compose ps` and inspect the Ollama/model-init logs. Wait for the model pull to complete.
- If the computer runs out of memory or replies are too slow, try a smaller Ollama model and update both model references in Compose.
- The app can still perform database screening when Ollama is unavailable; chatbot requests will show an error until the model is ready.

## Important licensing and medical limitations

- DDInter 2.0 data is published under CC BY-NC-SA 4.0 terms; this may restrict commercial use. Review the upstream dataset and code licenses and provide required attribution before use or redistribution. The application code and dataset can have different licenses.
- A missing interaction row is a knowledge gap, not evidence that a drug combination is safe. Coverage and matching are incomplete; supplements, food, alcohol, dose, timing, patient conditions, and other factors may be absent.
- The chatbot explains screening output and can hallucinate. It must not be relied on to decide that a combination is safe or to start, stop, or change medication. Confirm decisions with a pharmacist or prescriber. For suspected overdose or severe symptoms, contact emergency services or poison control.
- Chat inference runs in the local Ollama container. The optional RxNav fallback sends unmatched medication names to the RxNav service for name matching. Do not enter patient-identifying information.

## Attributions

OpenMRS Drug-Drug Interaction Knowledge Base:
https://github.com/pbiondich/openmrs-ddi-knowledge-base/tree/main


## Application Overview

### Drug Interaction Responses

![Drug interaction responses](/images/results.png)

### Chatbot explanation of results

![Chatbot explanation](/images/chatbot.png)

### Chatbot explanation simplified

![Chatbot simplified](/images/chatbot-simplified.png)