# Knowledge Collaboratory

[![Test production API](https://github.com/MaastrichtU-IDS/knowledge-collaboratory/actions/workflows/test-prod.yml/badge.svg)](https://github.com/MaastrichtU-IDS/knowledge-collaboratory/actions/workflows/test-prod.yml) [![Run backend tests](https://github.com/MaastrichtU-IDS/knowledge-collaboratory/actions/workflows/test-backend.yml/badge.svg)](https://github.com/MaastrichtU-IDS/knowledge-collaboratory/actions/workflows/test-backend.yml) [![CodeQL analysis](https://github.com/MaastrichtU-IDS/knowledge-collaboratory/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/MaastrichtU-IDS/knowledge-collaboratory/actions/workflows/codeql-analysis.yml)

Services to query the Nanopublications network using Translator standards to retrieve the Knowledge Collaboratory graph, a collection of drug indications annotated using preferred identifiers (usually from MONDO, CHEBI, DrugBank, etc).

A website to enable users to easily annotate, publish and browse biomedical claims from natural language to a structured format recognized by the Translator. To publish new claims the users login with their ORCID, and submit the sentence they want to annotate, optionally providing an additional link to specify the provenance of this statement. 

The model for annotating drug indications claims built for the LitCoin competition is used to automatically extract potential entities and relations. The SRI NameResolution API is then used to retrieve standard identifiers for each entities. And the BioLink model is used to define the relations between entities.

The extracted entities and relations are then displayed to the users on the website, and users can change the automatically generated claim to better reflect the emitted statement before publishing it to the Nanopublication network.

Backend built with [FastAPI](https://fastapi.tiangolo.com/), and [RDFLib](https://github.com/RDFLib/rdflib).

Frontend built with [React](https://reactjs.org) and [Material UI](https://mui.com/)

## 📥️ Requirements

* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/install/)

* [Poetry](https://python-poetry.org/) for backend development
* [Node.js](https://nodejs.org/en/) (with `npm`) and [`yarn`](https://yarnpkg.com/) if you need to do frontend development

## 🚀 Production deployment 

Checkout the `docker-compose.prod.yml` file for more details about the deployment.

1. Create a `.env` file with your production settings:

```
ORCID_CLIENT_ID=APP-XXX
ORCID_CLIENT_SECRET=XXXX
FRONTEND_URL=https://collaboratory.semanticscience.org
```

2. Deploy the app with production config: 

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## 🐳 Local development with docker

### Prepare the data

1. Create a `.env` file with your development settings in the root folder of this repository (you can copy `.env.sample`):

```bash
ORCID_CLIENT_ID=APP-XXX
ORCID_CLIENT_SECRET=XXXX
FRONTEND_URL=http://localhost:19006
```

2. Download and unzip the NER models in the `/data/knowledge-collaboratory/ner-models` folder:

```bash
mkdir -p /data/knowledge-collaboratory/ner-models
cd /data/knowledge-collaboratory/ner-models
wget https://download.dumontierlab.com/ner-models/litcoin-ner-model.zip
wget https://download.dumontierlab.com/ner-models/litcoin-relations-extraction-model.zip
unzip "*.zip"
rm *.zip
```

3. Clone and install the custom `react-taggy` library in `/opt` 

```bash
cd /opt
git clone --branch add-onclick-callback https://github.com/vemonet/react-taggy.git
yarn
```

### Start the stack

Start the backend with Docker Compose from the root folder of this repository:

```bash
docker-compose up -d
```

Then start the react frontend:

```bash
cd frontend/app
yarn
yarn dev
```

Now you can open your browser and interact with these URLs:

* Automatic OpenAPI documentation with Swagger UI: http://localhost/docs
* Alternative OpenAPI documentation with ReDoc: http://localhost/redoc
* GraphQL endpoint with Strawberry: http://localhost/graphql
* Frontend: http://localhost:19006

If you need to completely reset the Python cache:

```bash
docker-compose down
sudo rm -rf **/__pycache__
docker-compose build --no-cache
```

## ✅ Tests

2 sets of tests are available: `integration` tests to test local changes, and `production` tests to test the API deployed in production

You can run the tests in docker when the backend is already running: 

```bash
docker-compose exec backend poetry run pytest tests/integration -s
```

Or locally directly with poetry:

```bash
poetry install
poetry run pytest tests/integration -s
```

## 🔧 Maintenance

### 📦️ Add new packages

By default, the dependencies for the backend are managed with [Poetry](https://python-poetry.org/), install it if necessary.

In the `backend` folder you can install all the dependencies with:

```bash
poetry install
```

To add a new dependencies, run:

```bash
poetry add my-package@^1.0.0
```

You can also directly change the `pyproject.toml` file, and update the installed dependencies with:

```bash
poetry update
```

> If you don't have poetry installed locally or are facing issue with it, you can also add new packages with `docker-compose`, while the docker-compose is running run:
>
> ```bash
> docker-compose exec backend poetry add my-package
> ```

If you install a new package you will need to stop the current docker-compose running, then restarting it to rebuild the docker image:

```bash
docker-compose up --build --force-recreate
```

You can start a shell session with the new environment with:

```bash
poetry shell
```

Next, open your editor at `./backend/` (instead of the project root: `./`), so that you see an `./app/` directory with your code inside. That way, your editor will be able to find all the imports, etc. Make sure your editor uses the environment you just created with Poetry.

During development, you can change Docker Compose settings that will only affect the local development environment, in the file `docker-compose.override.yml`

### ⏫ Upgrade TRAPI version

Get the latest TRAPI YAML: https://github.com/NCATSTranslator/ReasonerAPI/blob/master/TranslatorReasonerAPI.yaml

For the OpenAPI specifications: change the `TRAPI_VERSION_TEST` in `backend/app/config.py`

For the reasoner_validator tests:

1. Change `TRAPI_VERSION_TEST` in `backend/app/config.py`

2. In `pyproject.toml` upgrade the version for the [reasoner-validator](https://pypi.org/project/reasoner-validator/), and update the dependencies locally:

```bash
poetry update
```

3. Run the tests:

```bash
poetry run pytest tests/integration -s
```

## 🐳 Docker Compose files and env vars

There is a main `docker-compose.yml` file with all the configurations that apply to the whole stack, it is used automatically by `docker-compose`.

And there's also a `docker-compose.override.yml` with overrides for development, for example to mount the source code as a volume. It is used automatically by `docker-compose` to apply overrides on top of `docker-compose.yml`.

These Docker Compose files use the `.env` file containing configurations to be injected as environment variables in the containers.

They also use some additional configurations taken from environment variables set in the scripts before calling the `docker-compose` command.

It is all designed to support several "stages", like development, building, testing, and deployment. Also, allowing the deployment to different environments like staging and production (and you can add more environments very easily).

They are designed to have the minimum repetition of code and configurations, so that if you need to change something, you have to change it in the minimum amount of places. That's why files use environment variables that get auto-expanded. That way, if for example, you want to use a different domain, you can call the `docker-compose` command with a different `DOMAIN` environment variable instead of having to change the domain in several places inside the Docker Compose files.

Also, if you want to have another deployment environment, say `preprod`, you just have to change environment variables, but you can keep using the same Docker Compose files.

## 🔗 Links

Livestream logs:

* https://fastapi.tiangolo.com/advanced/websockets/
* https://amittallapragada.github.io/docker/fastapi/python/2020/12/23/server-side-events.html

Project bootstrapped with https://github.com/tiangolo/full-stack-fastapi-postgresql

## 🗃️ Current resources in the Collaboratory

- A dataset of 1971 drug indications retrieved from the [PREDICT publication](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3159979/) and used as reference dataset for the [OpenPredict model](https://github.com/MaastrichtU-IDS/translator-openpredict)
  - Published in Nanopub Index http://purl.org/np/RAWWaT9M_Nd8cVm_-amJErz60Ak__tkS6ROi2P-swdmMw
  - Containing the following subsets:
    - http://purl.org/np/RAVQGOE6nCeX6Tp1rqBOKleQElWhgb1PGHvbIvH1TCE1A
    - http://purl.org/np/RA556Vv9lmc7jqKZkRV_5CP8Xz4GZaVbgUdIsye7tynN0
- 324 [Off-label drug indications](https://docs.google.com/spreadsheets/d/1fCykLEgAd2Z7nC9rTcW296KtBsFBBZMD8Yghcwv4WaE/edit#gid=428566902) found in PubMed publications
  - Published in Nanopub Index http://purl.org/np/RAaZp4akBZI6FuRzIpeksyYxTArOtxqmhuv9on-YssEzA
- Drug indications added by users (mainly from the default set of DailyMed indications proposed)
