
## 环境准备

 - Docker 安装 Meilisearch

    ```bash
    # Fetch the latest version of Meilisearch image from DockerHub
    docker pull getmeili/meilisearch:v1.9

    # Launch Meilisearch in development mode with a master key
    docker run -it --rm \
        -p 30019:7700 \
        -e MEILI_ENV='development' \
        -e MEILI_MASTER_KEY='xxxxxxxxxxxx' \
        -v ${pwd}/meili_data:/meili_data \
        getmeili/meilisearch:v1.9
    # Use ${pwd} instead of $(pwd) in PowerShell
    ```

    ```bash title="craete index"
    curl \
        -X POST 'http://localhost:30019/indexes' \
        -H 'Content-Type: application/json' \
        -H 'Authorization: Bearer xxxxxxxxxxxx' \
        --data-binary '{
            "uid": "movies",
            "primaryKey": "id"
        }'
    ```

    ```bash title="add or replace documents"
    curl \
        -X POST 'http://localhost:30019/indexes/movies/documents' \
        -H 'Content-Type: application/json' \
        -H 'Authorization: Bearer xxxxxxxxxxxx' \
        --data-binary '[
            {
            "id": 287947,
            "title": "Shazam",
            "poster": "https://image.tmdb.org/t/p/w1280/xnopI5Xtky18MPhK40cZAGAOVeV.jpg",
            "overview": "A boy is given the ability to become an adult superhero in times of need with a single magic word.",
            "release_date": "2019-03-23"
            }
        ]'
    ```
    