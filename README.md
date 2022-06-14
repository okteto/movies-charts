# movies-charts

Meta repo with charts

In order to push charts to the Okteto Registry, follow the following steps:


## Requirements

- Make sure your user has admin access to Okteto. This makes it possible to push to the `okteto` namespace in the Okteto Registry. Only admins can **push** to the `okteto` namespace, but all users will be able to **pull** from the `okteto` namespace.

- Install `helm` >= 3.8.0

## Steps

- Configure the Okteto CLI to your environment. Replace https://cloud.okteto.com by your okteto domain:
```
okteto context use https://cloud.okteto.com
```

- You can see your username, token and registry URL with `okteto context show`. We will refer to these values as  `<context.show.username>`,  `<context.show.token>`, and  `<context.show.registry>`.

- Configure `helm` to access the Okteto Registry:
```
echo <context.show.token> | helm registry login <context.show.registry> -u <context.show.username>  --password-stdin
```

- Create your helm package, in this case:
```
helm package api
```

```
Successfully packaged chart and saved it to: /Users/xxxx/xxxx/movies-charts/movies-api-0.1.0.tgz
```
- Push the chart to `okteto` namespace in the Okteto Registry:
```
helm push /Users/xxxx/xxxx/movies-charts/movies-api-0.1.0.tgz oci://<context.show.registry>/okteto
```

**Pro Tip**: you can automate these steps on every merge to keep your helm charts up to date in the Okteto Registry.
From this moment, you can configure any Okteto Pipeline to pull this chart and deploy it with the following commands:

```
- echo "${OKTETO_TOKEN}" | helm registry login <context.show.registry> -u ${OKTETO_USERNAME} --password-stdin
- helm pull oci://<context.show.registry>/okteto/movies-api --version 0.1.0
- helm upgrade --install movies-api oci://<context.show.registry>/okteto/movies-api --version 0.1.0 --set image=${OKTETO_BUILD_API_IMAGE}
```

You can see a sample of a pipeline using this approach [here](https://github.com/okteto/movies-api/blob/oci-chart/okteto.yml).

**Note**: `OKTETO_USER` and `OKTETO_TOKEN` are automatically available when deploying your git repo from the Okteto UI.
When running `okteto deploy` locally, your need to do:

```
export OKTETO_TOKEN=<context.show.token>
```

This is a limitation that will be fixed soon.
