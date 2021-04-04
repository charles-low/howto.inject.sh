Reference: https://stackoverflow.com/questions/31251356/how-to-get-a-list-of-images-on-docker-registry-v2

Listing repos of a registry

```
curl -X GET https://registry:5000/v2/_catalog
> {"repositories":["redis","ubuntu"]}
```

Listing tags of a repo

```
curl -X GET https://myregistry:5000/v2/ubuntu/tags/list
> {"name":"ubuntu","tags":["14.04"]}
```

With domain certificate and larger page size
```
curl --cacert domain.crt https://your.registry:5000/v2/_catalog?n=2000
```

List all repos and tags
```
REPO_URL=10.230.47.94:443

curl -k -s -X GET https://$REPO_URL/v2/_catalog \
 | jq '.repositories[]' \
 | sort \
 | xargs -I _ curl -s -k -X GET https://$REPO_URL/v2/_/tags/list
 ```
 
 Extract creds from .docker/config.json for CURL use
 ```
 CREDS=$(jq -r ".[\"auths\"][\"$THE_REGISTRY\"][\"auth\"]" .docker/config.json | base64 -d)
 curl -s --user $CREDS <URL>
  ```
  
 Pulling manifest of the image
 ```
 curl https://registry:5000/v2/image-name/manifests/latest
 ```
 
 Pulling blob for building FS
 ```
 curl -X GET http://192.84.233.3:5000/v2/treasure-trove/blobs/sha256:2a62ecb2a3e5bcdbac8b6edc58fae093a39381e05d08ca75ed27cae94125f935 --output 1.t
ar
```
```
curl -X GET http://192.84.233.3:5000/v2/treasure-trove/manifests/latest | jq -r '.fsLayers[].blobSum'
```
```
COUNT=0  curl -X GET http://192.84.233.3:5000/v2/treasure-trove/manifests/latest | jq -r '.fsLayers[].blobSum' |  while read -r BLOB; do echo "http://192.84.233.3:5000/v2/treasure-trove/blobs/${BLOB}"; curl -vv -X GET "http://192.84.233.3:5000/v2/treasure-trove/blobs/${BLOB}" --output ${COUNT}.tar ; let COUNT=COUNT+1; done
```
```
curl -X GET http://192.84.233.3:5000/v2/treasure-trove/manifests/latest | jq -r '.fsLayers[].blobSum' |  while read -r BLOB; do echo "http://192.84.233.3:5000/v2/treasure-trove/blobs/${BLOB}"; curl -vv -X GET "http://192.84.233.3:5000/v2/treasure-trove/blobs/${BLOB}" |tar -C output -xf - ; done
```