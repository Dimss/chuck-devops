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
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/s2i/bc.yaml
```


oc get templates jenkins-ephemeral -n openshift  -o yaml | oc process -pENABLE_OAUTH=false -f -  | oc create  -f -