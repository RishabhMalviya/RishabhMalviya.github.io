---
layout: page
title: ml-deployment-template
description: A Docker-based template for packaging an ML model as a REST API, with zero-downtime re-deployments via Traefik and a local dev environment that mirrors production exactly.
img: assets/img/projects/ml-deployment-template.png
importance: 3
github: https://github.com/RishabhMalviya/ml-deployment-template
---

---
*Read the Medium post for this project [here](https://medium.com/better-programming/create-a-zero-downtime-deployment-of-your-machine-learning-api-6486cb6394c3)*

---

Say you've worked hard to build and fine-tune a model — maybe for a Kaggle competition, maybe for your own portfolio. Now you want to make it available for everyone's use, but you're not entirely sure how to engineer it into an API. **This template project is meant to solve that problem.** It's built for anyone who wants to quickly and easily deploy a machine learning model as an API.

It's designed around Docker so that your development and deployment environments are identical, and it uses **Traefik** for zero-downtime re-deployments — which makes continuously updating the code behind a deployed model very smooth. Because everything is Docker-based, it translates easily into the CI/CD pipeline of your choice; the [`docker-compose.yml`](https://github.com/RishabhMalviya/ml-deployment-template/blob/master/docker-compose.yml) already has configurations for two basic stages, testing and deployment.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        <img src="{{ 'assets/video/ml-deployment-template.gif' | relative_url }}" alt="Zero-downtime deployment in action" class="img-fluid rounded z-depth-1" loading="lazy" />
    </div>
</div>
<div class="caption">
    Zero-downtime deployment in action — the old container keeps serving until the new one is healthy.
</div>

I wrote up the deployment infrastructure in detail in [this Medium post](https://medium.com/better-programming/create-a-zero-downtime-deployment-of-your-machine-learning-api-6486cb6394c3).

## Adding your code

The default functionality already in the project — sentence comparison with a BERT sentence encoder — is there as an example to mimic.

1. Inject your ML code into the `./model` directory.
2. Specify your endpoints in `app.py`. The sentence comparison functionality is exposed from [this endpoint](https://github.com/RishabhMalviya/ml-deployment-template/blob/master/app.py#L109); there's also [a dummy endpoint](https://github.com/RishabhMalviya/ml-deployment-template/blob/master/app.py#L56) for quick testing.
3. Optionally, write the corresponding test cases in `./api/tests/test_app.py`.

## Deploying to your server

1. Make sure Docker, docker-compose, and git are installed on your Linux server.
2. SSH in and clone your code.
3. Run `deploy.sh` from the root of the cloned repository.

That's literally it. You can re-deploy whenever you want, with zero downtime, by re-running that script.

## Local development environment

One of the most useful things about this template is that you can spin up a local development environment that mimics your deployment exactly — again, thanks to Docker and docker-compose.

1. Create a `.env` file with the appropriate values (see `.env.template`). It's gitignored by default, so your local values never reach the remote repo.
2. Run `docker-compose up -d recommender-dev`.
3. Get a shell in the dev container with `docker exec -it recommender-dev bash`. You can now use it just like a terminal on your local machine.

From inside the container:

```bash
# Bring up the API
uvicorn --host 0.0.0.0 --port 80 app:app

# Run the unit testing suite
python3 -m pytest

# Run a Jupyter notebook (without authentication)
jupyter notebook --ip 0.0.0.0 --port 5000 --allow-root \
  --NotebookApp.token='' --NotebookApp.password=''
```

You'll want to connect your editor to the interpreter at `/usr/bin/python3` inside the `recommender-dev` container — supported by PyCharm Professional and by VS Code with the Remote Containers extension.

### Debugging API calls

The code under the main-guard (`if __name__ == "__main__"`) in `app.py` brings up the API, creates a test client, and sends a request to it — so any breakpoints you've set in your request-handling code will be hit.
