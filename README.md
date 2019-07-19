# CI/CD with OCP/OKD

### Setup environment
```bash
# Application project 
oc new-project chuck
# DevOps project 
oc new-project chuck-ops
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
# Start Build
oc start-build chuck-ui-custom -F
# Cleanup istags 
oc get istag -n chuck | grep chuck-ui  | awk '{print $1}' | xargs oc delete istag -n chuck
# Cleanup BC 
oc delete bc chuck-ui-custom -n chuck-ops

```

### Pipeline build strategy
```bash
# Deploy Jenkins
 oc get templates jenkins-ephemeral -n openshift  -o yaml | oc process -pENABLE_OAUTH=false -pMEMORY_LIMIT=2048Mi -f -  | oc create  -f - -n chuck-ops
# Login to Jenkins: admin/password
https://jenkins-chuck-ops.router.default.svc.cluster.local:9443/login
# Create DC
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/pipeline/bc.yaml
# Start build from Jenkins UI and from CLI
oc start-build chuck-ui-pipeline
# Cleanup BC 
oc delete -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/pipeline/bc.yaml
# Cleanup Jenkins
oc get templates jenkins-ephemeral -n openshift  -o yaml | oc process -pENABLE_OAUTH=false -pMEMORY_LIMIT=2048Mi -f -  | oc delete  -f - -n chuck-ops
```


