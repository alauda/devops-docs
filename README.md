# DevOps docs

## Dependencies / Setup

Visit [DEVELOPMENT.md](DEVELOPMENT.md) for more information.

## Writing and translation

All new content should be written in English inside `docs/en` and translated automatically to chinese using `yarn translate` command below:

```
export AZURE_OPENAI_API_KEY=<your-api-key>
yarn translate -s en -t zh -g '*' --copy
```

> **Note**: Contact @yongsong for API key

After translation, you can preview the changes by running:

```
yarn dev
```

## Build

After merging docs changes into main, it is necessary to check the build pipeline to ensure the docs are built successfully here: 

- [Build pipelines](https://edge.alauda.cn/console-devops/workspace/alauda/ci?namespace=alauda-dev&cluster=business-build&buildName=doc-build-devops)
