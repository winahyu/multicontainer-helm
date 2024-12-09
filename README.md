Guided Exercise: Install Applications by Using Helm Charts
Install multi-container applications by using Helm Charts.

Outcomes


Manage and deploy multi-container applications with Helm.


As the student user on the workstation machine, use the lab command to prepare your environment for this exercise, and to ensure that all required resources are available.

[student@workstation ~]$ lab start multicontainer-helm
Instructions

Log in to RedÂ Hat OpenShift.

Log in to OpenShift as the developer user.
```
[student@workstation ~]$ oc login -u developer -p developer \
https://api.ocp4.example.com:6443
Login successful.
...output omitted...
Ensure that you use the multicontainer-helm project.

[student@workstation ~]$ oc project multicontainer-helm
Already on project "multicontainer-helm" on server "https://api.ocp4.example.com:6443".
Modify the expense-service helm chart to use the expenseService.image variable.

Change to the ~/DO288/labs/multicontainer-helm/expense-service directory.

[student@workstation ~]$ cd ~/DO288/labs/multicontainer-helm/expense-service
Configure the default expenseService.image variable in the values.yaml file.

...file omitted...
```
expenseService:
  replicaCount: 1
  image: "registry.ocp4.example.com:8443/redhattraining/ocpdev-expense-service:4.14"
  host: "TODO"
  domain: "apps.ocp4.example.com"
Configure the templates/expense-deployment.yaml template to use the expenseService variable. Wrap the file in the with block.

{{ with .Values.expenseService }}
apiVersion: apps/v1
kind: Deployment
metadata:
...file omitted...
        - containerPort: 8443
          protocol: TCP
{{ end }}
Set the .spec.replicas property.

...file omitted...
spec:
  replicas: {{ .replicaCount }}
  selector:
    matchLabels:
      deployment: expense-service
...file omitted...
Set the .spec.template.spec.containers.image property.

...file omitted...
            valueFrom:
              secretKeyRef:
                key: database-user
                name: postgresql
        image: {{ .image | quote }}
        name: expense-service
...file omitted...
Save your changes.

Test that the templates/expense-deployment.yaml template renders correctly.

[student@workstation expense-service]$ helm template \
-s templates/expense-deployment.yaml .
---
# Source: expense-svc/templates/expense-deployment.yaml
apiVersion: apps/v1
kind: Deployment
...output omitted...
spec:
  replicas: 1
...output omitted...
        image: "registry.ocp4.example.com:8443/redhattraining/ocpdev-expense-service:4.14"
        name: expense-service
...output omitted...
Configure the expense-service route to use the expenseService.host property.

Configure the default expenseService.host variable in the values.yaml file.

...file omitted...

expenseService:
  replicaCount: 1
  image: "registry.ocp4.example.com:8443/redhattraining/ocpdev-expense-service:4.14"
  host: "expense-service"
  domain: "apps.ocp4.example.com"
Configure the templates/expense-route.yaml template to use the expenseService.host variable.

...file omitted...
spec:
  host: {{ .Values.expenseService.host }}.{{ .Values.expenseService.domain }}
  port:
    targetPort: http
  to:
    kind: ""
    name: expense-service
Save your changes.

Test that the templates/expense-route.yaml template renders correctly.

[student@workstation expense-service]$ helm template \
-s templates/expense-route.yaml .
---
...output omitted...
  name: expense-service
spec:
  host: expense-service.apps.ocp4.example.com
  port:
...output omitted...
Configure the database-password property in the templates/postgres-secret.yaml template.

Use the postgres.pass property if it is present in the values.yaml file. Otherwise, generate a random 20-character alphanumeric string as the password.

Encode strings in the Secret resource as base64.

Open the templates/postgres-secret.yaml template and add the database-password property.

...file omitted...
  database-user: {{ randAlphaNum 20 | b64enc }}
  {{end}}
  {{ if .Values.postgres.pass }}
  database-password: {{ .Values.postgres.pass | b64enc }}
  {{else}}
  database-password: {{ randAlphaNum 20 | b64enc }}
  {{end}}
Save your changes.

Test that the templates/postgres-secret.yaml template renders correctly.

[student@workstation expense-service]$ helm template \
-s templates/postgres-secret.yaml .
...output omitted...
  name: postgresql
data:
  database-name: c2FtcGxlZGI=

  database-user: NUdHQ2s3VnZxMk0yc2JWdnhxeGQ=


  database-password: bldhYTRtaFpzUFd1ZDFZUGMxSDk=
Configure the message that users see after using the helm chart.

Add the URL of the deployed application to the output message.

Open the templates/NOTES.txt file and add the following text:

...file omitted...
Your release is named {{ .Release.Name }} deployed in {{ .Release.Namespace }}.

Your application is available at: "{{ .Values.expenseService.host }}.{{ .Values.expenseService.domain }}/expenses"

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}
Note
Write the "{{ .Values.expenseService.host }}.{{ .Values.expenseService.domain }}/expenses" template in a single line to produce the correct output.

Deploy the Helm chart and test the application.

Create an expense-service Helm release.

[student@workstation expense-service]$ helm install --wait \
expense-service .
...output omitted...
Thank you for installing expense-service.

Your release is named expense-service deployed in multicontainer-helm.

Your application is available at: "expense-service.apps.ocp4.example.com/expenses"
...output omitted...
Test the application.

[student@workstation expense-service]$ curl \
expense-service.apps.ocp4.example.com/expenses | jq
[
  {
    "id": 5,
    "amount": 15.00,
    "associateId": 1,
    "name": "Phone",
    "paymentMethod": "CASH",
    "uuid": "499053c1-bae6-d40c-24d7-463134d08bec"
  },
...output omitted...
Finish

On the workstation machine, use the lab command to complete this exercise. This step is important to ensure that resources from previous exercises do not impact upcoming exercises.

[student@workstation ~]$ lab finish multicontainer-helm
Revision: do288-4.14-9c142f1
