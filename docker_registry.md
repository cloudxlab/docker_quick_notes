Create your own account on Docker registry hub.

https://hub.docker.com/


You first login to your hub account.
```
docker login
```

You can serach the image in repo.
```
docker search manojkmhub/flaskapp
docker search ubuntu
```

List all tags from remote registry for a givem image.
```
curl -s -S "https://registry.hub.docker.com/v2/repositories/library/ubuntu/tags/" | jq '."results"[]["name"]' |sort

wget -q https://registry.hub.docker.com/v1/repositories/debian/tags -O -  | sed -e 's/[][]//g' -e 's/"//g' -e 's/ //g' | tr '}' '\n'  | awk -F: '{print $3}'
```

**Pushing your own images to registry hub**

When you do above, your password will be stored unencrypted in `/home/ubuntu/.docker/config.json`.

Then, you tag your locallly created image to your account.
```
docker tag flaskapp:latest manojkmhub/flaskapp:latest
```

Then, you push the image to your hub account.
```
docker push manojkmhub/flaskapp:latest
```


