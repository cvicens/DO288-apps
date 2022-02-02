
podman build -t s2i-do288-go .

mkdir /tmp/s2i-go-app

s2i build ./test/test-app s2i-do288-go s2i-go-app --as-dockerfile /tmp/s2i-go-app/Containerfile

ls /tmp/s2i-go-app/

podman build -t s2i-go-app /tmp/s2i-go-app/

podman images | grep s2i

podman run --name go-test -u 1234 -p 8080:8080 -d s2i-go-app

curl http://localhost:8080/carlos

podman stop go-test && podman rm go-test

oc new-project $USERNAME-custom-s2i

oc create secret docker-registry quayio \
--docker-server quay.io \
--docker-username cvicens \
--docker-password yourpassword

oc secrets link builder quayio

podman login quay.io

skopeo copy \
containers-storage:localhost/s2i-do288-go \
docker://quay.io/${RHT_OCP4_QUAY_USER}/s2i-do288-go

oc import-image s2i-do288-go --from=quay.io/cvicens/s2i-do288-go --confirm=true

oc new-app --name greeter --context-dir=/go-hello s2i-do288-go~https://github.com/cvicens/DO288-apps

oc expose svc/greeter

curl http://$(oc get route/greeter -o jsonpath='{.spec.host}')/cvicens