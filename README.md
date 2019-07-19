# CI/CD with OCP/OKD

### Setup environment
```bash
# Application project 
oc create -f chuck
# DevOps project 
oc create -t chuck-ops
# Allow builder user to push IS in chuck project
oc policy add-role-to-user system:image-pusher system:serviceaccount:chuck-ops:builder --namespace=chuck
# Create IS for Chuck-UI
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/is.yaml
# Create DC for API service
oc create -f https://raw.githubusercontent.com/Dimss/chuck-api/master/ocp/cd.yaml
# Create DC for UI service
oc create -f https://github.com/Dimss/chuck-ui/blob/master/ocp/cd.yaml
```


### S2I build strategy
```bash
oc create -f https://raw.githubusercontent.com/Dimss/chuck-devops/master/ci/s2i/bc.yaml
```


oc get templates jenkins-ephemeral -n openshift  -o yaml | oc process -pENABLE_OAUTH=false -f -  | oc create  -f -