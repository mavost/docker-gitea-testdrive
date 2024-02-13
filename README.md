# A Gitea testing suite

*Date:* 2024-02-09  
*Author:* MvS  
*keywords:* GIT, code repository, PostgreSQL

## Description

Gitea is a self-hosted Git server that allows teams to collaborate on both open-source and
private projects. It can be used as an alternative to GitHub.
The backend is based on a standalone PostgreSQL instance.
This test suite uses docker-compose, `Makefile` and an `.env` file functionality for customization.

### Initial configuration of Gitea

We assume to run a standalone DB instance and another Gitea instance in a stack with named Docker volumes, i.e.,
tied to a user on the host system.

1. We will use a customized Gitea configuration, so copy the `config/app.alt` to a `config/app.ini`
file and edit the configuration according to the [cheat sheet](https://docs.gitea.com/administration/config-cheat-sheet).
Note that database connectivity and server routing will have to match the settings from, e.g., Postgres backend
and Traefik reverse proxy.
2. (Optional): enable push-create of repositories from pre-existing local repositories

    ```bash
    #edit in config/app.ini
    [repository]
    ENABLE_PUSH_CREATE_USER = true
    ```

3. Copy the `.env-dist` to a `.env` file and edit it to match you host username and cvs endpoint.
4. Run `docker compose up`.
5. (Optional, depending on config) Navigate to `localhost:8300` via browser and click *install*.
6. Another login page will appear where you can sign up with a new git user which will become *admin*.
7. test