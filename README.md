# Jupyter Notebook with Ruby Kernel & NLP Gems

[![CI](https://github.com/b08x/jupyter-ruby-docker/actions/workflows/ci.yml/badge.svg)](https://github.com/b08x/jupyter-ruby-docker/actions/workflows/ci.yml)

Based on RubyData's [docker-stacks](https://github.com/RubyData/docker-stacks) and the official [Jupyter Docker Stacks](https://github.com/jupyter/docker-stacks).

<div style="float: left; margin-right: 20px;">
  <img src="docs/assets/img/info_graphic.webp" alt="Process Diagram" width="500" height="500">
</div>
This project provides Docker images that bundle a Jupyter Notebook server preconfigured with a Ruby kernel (IRuby) and a comprehensive suite of RubyGems focused on Natural Language Processing (NLP) and language model utilities.

## Features

- **Ruby Kernel for Jupyter**: Use IRuby to run Ruby code interactively within Jupyter notebooks.
- **NLP Libraries**: Includes gems for text tokenization, sentiment analysis, entity extraction, and various language utilities.
- **Cross-Language Integration**: Leverage Python libraries (like spaCy, NLTK, Transformers) from Ruby via [pycall](https://rubygems.org/gems/pycall) for mixed-language workflows.
- **Data Analysis**: Utilize gems like [daru](https://rubygems.org/gems/daru) and [daru-view](https://rubygems.org/gems/daru-view) for data manipulation and visualization.
- **Vector Database Integration**: Includes `pgvector` and `redis` services via Docker Compose for vector similarity searches.

## Prerequisites

- [Docker](https://www.docker.com/get-started) installed on your machine.
- [Docker Compose](https://docs.docker.com/compose/install/) (Recommended for full functionality).
- Basic familiarity with Jupyter Notebooks and Docker commands.

## Building the Images

The project contains two primary Docker images: `base` and `nlp`.

- **`base`**: A foundational image built upon `jupyter/docker-stacks-foundation`, including JupyterLab, common Python data science libraries, and essential NLP tools like spaCy and Google Generative AI SDK.
- **`nlp`**: Built upon the `base` image, this adds Ruby, IRuby kernel, and a wide range of Ruby NLP gems (see `nlp/Gemfile`).

1. **Clone the Repository**

    ```bash
    git clone https://github.com/b08x/jupyter-ruby-docker.git
    cd jupyter-ruby-docker
    ```

2. **Build the Images**

    You can build the images individually using `docker build` or use the provided `Rakefile`:

    - **Using Docker Build (NLP image, depends on base):**

        ```bash
        # Build the base image first (if not pulled from a registry)
        docker build -t b08x/notebook-base:latest -f base/Dockerfile .

        # Build the NLP image
        docker build -t b08x/notebook-nlp:latest -f nlp/Dockerfile .
        ```

    - **Using Rake (Recommended):**

        ```bash
        # Build the nlp image (will build base automatically if needed)
        rake build/nlp

        # Or build all defined images
        rake build-all
        ```

## Running the Container

You can run the `nlp` image using either `docker run` or `docker-compose`. Using `docker-compose` is recommended as it also sets up the `redis` and `pgvector` services defined in `docker-compose.yml`.

### Using `docker run` (Notebook only)

This method starts only the Jupyter notebook container.

```bash
docker run --rm -p 8888:8888 \
  -v "${PWD}/work":/home/jovyan/work \
  --user $(id -u):$(id -g) \
  b08x/notebook-nlp:latest
```

**Explanation:**

- `--rm`: Automatically remove the container when it exits.
- `-p 8888:8888`: Map port 8888 on your host to port 8888 in the container.
- `-v "${PWD}/work":/home/jovyan/work`: Mount a local directory named `work` into the container's home directory. **Create this directory (`mkdir work`) in your project folder first.** This allows you to persist your notebooks and data.
- `--user $(id -u):$(id -g)`: Run the container using your current user and group ID to avoid permission issues with mounted volumes.
- `b08x/notebook-nlp:latest`: The image to run.

**Access Jupyter:** Open your browser and go to `http://localhost:8888` (or `http://<your-docker-ip>:8888`). The startup logs in your terminal will display an authentication token—use it to log in.

### Using `docker-compose` (Recommended: Notebook + Redis + Pgvector)

This method uses the `docker-compose.yml` file to start the `nlp-notebook`, `redis`, and `pgvector` services together.

1. **Environment Setup:**
    Copy the example environment file and adjust if necessary (especially if your user/group ID is not 1000):

    ```bash
    cp .env.example .env
    # Optional: Edit .env if your UID/GID is not 1000
    # You can find your UID/GID with: id -u and id -g
    ```

2. **Create Work Directory:**
    Ensure the directory you plan to mount exists:

    ```bash
    mkdir -p ~/Workspace # Or the path specified in docker-compose.yml volumes
    mkdir -p ~/Documents # Or the path specified in docker-compose.yml volumes
    ```

    *Note: The default `docker-compose.yml` mounts `~/Workspace` to `/home/jovyan/work` and `~/Documents` to `/home/jovyan/Library` (read-only).*

3. **Start Services:**

    ```bash
    docker-compose up -d
    ```

    - `-d`: Run containers in detached mode (in the background).

4. **Access Jupyter:**
    Open your browser to `http://localhost:8888`. Use the token from the logs (`docker-compose logs nlp-notebook`).

5. **Access RedisInsight:**
    Open your browser to `http://localhost:8001`.

6. **Stopping Services:**

    ```bash
    docker-compose down
    ```

## Usage Examples

- **IRuby Notebooks:** Within Jupyter, create a new notebook and select the Ruby 3.2 kernel (IRuby). You can then run Ruby code and take advantage of the installed NLP gems.
- **Integrate Python:** With [pycall](https://rubygems.org/gems/pycall) installed, you can call Python libraries (e.g., spaCy, NLTK) from your Ruby code.
- **NLP Pipelines:** Use gems such as [ruby-spacy](https://rubygems.org/gems/ruby-spacy) and [langchainrb](https://rubygems.org/gems/langchainrb) to build language processing pipelines and experiment with language models.
- **Vector Search:** Utilize the `pgvector` or `redis` services (when using `docker-compose`) for storing and querying vector embeddings.

## Overview of Included RubyGems

Key RubyGems included in the `nlp/Gemfile`:

- **iruby**: Provides the Ruby kernel for Jupyter.
- **pycall**: Allows Ruby code to call Python functions and libraries.
- **ruby-spacy**: Ruby wrapper for spaCy NLP library.
- **langchainrb**: Utilities for building language model workflows.
- **daru & daru-view**: Data analysis and visualization.
- **pgvector**: Ruby client for the Pgvector PostgreSQL extension.
- **redis**: Ruby client for Redis.
- **chroma-db**: Ruby client for ChromaDB vector store.
- **fuzzy_tools & pragmatic_tokenizer**: Text processing utilities.

For a complete list, review the [nlp/Gemfile](./nlp/Gemfile).

## Customization & Development

- **Add/Update RubyGems**: Modify `nlp/Gemfile` and rebuild the `nlp` image (`rake build/nlp` or `docker build -f nlp/Dockerfile ...`).
- **Add/Update Python Packages**: Modify `base/Dockerfile` (specifically the `pip install` or `mamba install` commands) and rebuild both `base` and `nlp` images.
- **System Packages**: Add `apt-get install` commands to the relevant `Dockerfile` (`base/Dockerfile` for Python/base dependencies, `nlp/Dockerfile` for Ruby/runtime dependencies) and rebuild.
- **Configuration**: Modify Jupyter configurations in `base/jupyter_server_config.py` or startup scripts (`base/start-notebook.py`, etc.) and rebuild.
- **Kernel Patching**: The `nlp/Dockerfile` applies a patch to `ruby-spacy` (via `respond_to_missing.patch`)—customize this patch if needed.

## Troubleshooting

- **Build Issues**: Verify build dependencies and Docker daemon resources. Check build logs for errors.
- **Kernel Registration**: If the Ruby kernel doesn't appear, ensure `iruby register --force` ran successfully during the `nlp` image build (`docker logs <container_id>` or check build output).
- **Port Conflicts**: If port 8888 (or others like 6379, 5432) is in use, modify the `docker run -p` mapping or the `ports` section in `docker-compose.yml`.
- **Volume Permissions**: If you encounter permission errors with mounted volumes, ensure the `--user $(id -u):$(id -g)` flag is used with `docker run` or that the `UID`/`GID` in your `.env` file matches your host user when using `docker-compose`.

## License

This project is licensed under the MIT License. See the LICENSE file (if available) or assume MIT if missing. Base images used may have their own licenses (e.g., BSD for Jupyter Docker Stacks).

## Contributing

Contributions are welcome! Please fork the repository and submit pull requests.
