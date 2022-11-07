# copy-image-manifest-action

Copy a image manifest from one registry to the other or 'retag' an existing manifest by using [regclient/regclient](https://github.com/regclient/regclient/).
Useful to retag your sha hash builds to tags without the need to rebuild.

## Option

| Name               | Type     | Description                 | Example |
|--------------------|----------|-----------------------------|---------|
| `source`           | String   | The manifest/image to retag                                       | `lansible/test:86b04b3b8f1d86d9d3c8322591d1e013a294f6f0`
| `targets`          | List/CSV | The targets to push the manifest to, seperated by commas          | `lansible/test:latest,ghcr.io/lansible/test:latest`
| `wait`             | Bool     | Wait on the source to be available (default `true`)               | `false`
| `wait_platforms`   | List/CSV | The platforms to wait for to be available (default `linux/amd64`) | `linux/amd64,linux/arm64`


## Example

Used in my workflow, the most up to date example:
[LANsible/github-workflows](https://github.com/LANsible/github-workflows/blob/main/.github/workflows/docker-build.yml)

```yaml
  retag_tag:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Login to GHCR.io
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Get tag name
        run: echo "tag_name=${GITHUB_REF##*/}" >> $GITHUB_ENV

      - name: Retag to tag and latest
        uses: LANsible/copy-image-manifest-action@main
        with:
          source: lansible/test:${{ github.sha }}
          targets: |
            lansible/test:{{ env.tag_name }}
            lansible/test:latest,
            ghcr.io/lansible/test:${{ env.tag_name }},
            ghcr.io/lansible/test:latest
          wait_platforms: linux/amd64,linux/arm64
```

## Credits

* [regclient/regclient](https://github.com/regclient/regclient/)
