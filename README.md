# Repo that reproduce a verification issue with blockscout

## Reproduction

clone it first and then install the dependencies

```bash
yarn
```

After that follow these steps:

1. in one terminal, run the node and deploy the contracts with this simple command :

```bash
yarn dev
```

2. in another terminal, run the docker that will instantiate blockscout and the postgres db

```bash
docker-compose up
```

(Note: the docker image used for blockscout is https://hub.docker.com/repository/docker/wighawag/blockscout)

It is built straight out of blockscout github repo at commit : cae5dd5c500555a7e111d015a164219a39ca0203 : https://github.com/blockscout/blockscout/tree/cae5dd5c500555a7e111d015a164219a39ca0203

See [here](#building-the-docker-image-for-blockscout) for instruction to build that docker image

3. finally in a third terminal, run the following

**PLEASE WAIT a bit so that blockscout get ready to process your request**

```bash
yarn verify localhost --api-url http://localhost:4000 --write-post-data --api-key None
```

## analysis

This third step will attempt to verify the 2 contracts that were deployed with `yarn dev`

One of them of them fail: `GreetingsRegistry`

and another succeed: `SimpleERC20`

The option `--write-post-data` is writing the request sent to blockscout api for debugging purpose. It generates 3 files per contract:

- `etherscan_requests/<network>/<contract-name>_multi-source.json` contains the json representing the source of the contract (same as the file to drop in the frontend ui)
- `etherscan_requests/<network>/<contract-name>.formdata` is the raw post data send to the api (using `'content-type': 'application/x-www-form-urlencoded'`)
- `etherscan_requests/<network>/<contract-name>.json` is the json representation of that post data

Note that both contract are verified without problem on Sokol network. Their api request are available in `etherscan_requests/sokol`

and you can see that the only difference is the addresses

I have no idea what is going on. There seems to be no flag to have more verbose output in blockscout and the log I see on the docker console bring no clue.

## building the docker image for blockscout

Here was the step to create that docker image

```bash
git clone git@github.com:blockscout/blockscout.git
cd blockscout
docker build  -f docker/Dockerfile . -t wighawag/blockscout:v0.0.1
docker tag wighawag/blockscout:v0.0.1 wighawag/blockscout:v0.0.1
docker push wighawag/blockscout:v0.0.1
```
