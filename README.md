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

### Migration of previous Gitea instance

#### Run backup of dockerized Postgres backend

1. Generate dump:  
`PGOPTIONS="-c statement_timeout=0" pg_dump -U gitea -h localhost -p 8432 -d giteadb > gitea-db.sql`

2. Preconditions for restore:

    - Rename old db to `giteadb_backup`
    - Let DB admin generate a new database as restore target:  
        - `psql -U <adminuser> -h localhost -p 8432 -d public`
        - Execute: `CREATE DATABASE giteadb OWNER gitea;`

3. Restore db from dump:  
`psql -U gitea -h localhost -p 8432 -d giteadb -f gitea-db.sql`

#### Run Backup of the docker volume containing the git repositories

1. Backup the volume of the existing Gitea instance to a file in current directory:  
`docker run --rm --volumes-from giteastack_gitea -v $(pwd):/backup busybox tar -czvf /backup/backup.tar.gz /data`

2. Create a new volume and an intermediate container for extraction.  
Note: keep in mind to use the standalone name of the volume in the `docker-compose.yaml`, i.e., `gitea_data`:  

    - `docker volume create gitea_data`
    - `docker create -v gitea_data:/data --name docker_extractor busybox true`

3. Untar the backup files into the new container᾿s data volume:  
`docker run --rm --volumes-from docker_extractor -v $(pwd):/backup busybox tar -xzvf /backup/backup.tar.gz`

4. Compare to the original container and remove intermediate container:  

    - `docker run --rm --volumes-from DATA busybox ls -la /data`
    - `docker rm docker_extractor`

#### Run deployment

1. Clone repository to new instance.
2. Duplicate `.env` file from original instance, copy it to repository root.
3. Edit `docker-compose.yaml`:
    - Disable `giteastack_gtpostgres` container and volume,
    - Change `giteastack_gitea` volume type to `external`,
    - Duplicate `app.ini` from original instance, copy it to `./config` and edit hostname.
4. Run `docker compose up`

### References

- [Rendering Jupyter notebooks](https://blog.gitea.com/render-jupyter-notebooks/)
- etc.
