# A gitea testing suite

*Date:* 2024-02-09  
*Author:* MvS  
*keywords:* GIT, code repository, postgreSQL

## Description

Gitea is a self-hosted Git server that allows teams to collaborate on both open-source and
private projects. It can be used as an alternative to GitHub.
The backend is based on a standalone postgreSQL instance.
This test suite uses docker-compose, `Makefile` and an `.env` file functionality for customization.

### Initial configuration of Gitea

We assume to run a standalone DB and Gitea instance in a stack with a Docker volumes
tied to a localhost user already existing on the system.

1. Copy the `.env-dist` to a `.env` file and edit it to match you host username.
2. Run `docker compose up`.
3. Navigate to `localhost:8300` via browser and click *install*.
4. Another login page will appear where you can sign up with a new git user which will become *admin*.
