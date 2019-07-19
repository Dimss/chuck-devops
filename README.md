# CI/CD with OCP/OKD

### Setup environment
```bash
# Application project 
oc new-project chuck
# DevOps project 
oc new-project chuck-ops
# Deploy Jenkins
oc get templates jenkins-ephemeral -n openshift  -o yaml | oc process -pENABLE_OAUTH=false -f -  | oc create  -f - -n chuck-ops
# Allow builder user to push IS in chuck project
oc policy add-role-to-user system:image-pusher system:serviceaccount:chuck-ops:builder --namespace=chuck
# Create IS for Chuck-UI
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/is.yaml
# Create DC for API service
oc create -f https://raw.githubusercontent.com/Dimss/chuck-api/master/ocp/cd.yaml
# Create DC for UI service
oc create -f https://raw.githubusercontent.com/Dimss/chuck-ui/master/ocp/cd.yaml
```

### Verify resources 
```bash
# Make sure API is running
oc get pods | grep chuck-api
# Exec request to API server
curl http://127.0.0.1:30081/v1/joke 
# Make sure IS for chuck-ui is exists
oc get is | grep chuck-ui
```


### S2I build strategy
```bash
# Create BC
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/s2i/bc.yaml
# Start build
oc start-build chuck-ui -F
# Cleanup - delete BC
oc delete bc chuck-u
# Cleanup - delete istag
oc delete istag chuck-ui:latest -n chuck
```

### Custom build strategy
```bash
# Create custom build config secret 
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/custom/secret.yaml
# Create BC 
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/custom/bc.yaml
```




