# Building historical releases

In order to make this as useful as possible, this repo also aims to build the historical builds.

```shell
gh api repos/simulot/immich-go/tags --paginate --jq '.[].name'
```
