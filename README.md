## Rancher QA pre-configured Jenkins backed by code (JCasC)

* Custom Jenkins image with plugins pre-installed (Chart: 5.15, Jenkins: 2.440.2-jdk)
* One configmap and secret backing the installation
* Docker-in-Docker agent w/ jenkins-job-builder installed
* Credentials automatically loaded from AWS Secrets Manager (see secrets)
* Admin token automatically configured (see secrets)
* `Build Jenkins Jobs` pre-configured

---

### To deploy:

* Build and push `jenkins` and `dind-agent` images
* Create namespace `jenkins`
* Adjust secrets in `resources.yaml`
  * `ADMIN-TOKEN` must be 32 characters  (ex: `114d13730ad0d9294e4b424bf8d873c26d`)
  * `AWS-SM-ACCESSKEY` & `AWS-SM-SECRETKEY` must belong to a user with `Read` privileges to AWS Secrets Manager
* Deploy `resources.yaml` to your cluster in `jenkins` namespace
* Add helm repo - `https://charts.jenkins.io`
* Deploy Jenkins chart to cluster with `values.yaml` - config assumes deployment name is `jenkins-qa`
* Deploy `ingress.yaml` after updating the host (`JENKINS_URL` with no protocol)
* Update Route 53 A record (`JENKINS_URL`) with the LB IPs from the ingress (`k get ingress -n jenkins`)
* Go to `JENKINS_URL`
* Login with admin credentials (set in secret)
* Run `Build Jenkins Jobs` job

---

### Some credentials in AWSSM are assumed to exist:

* `ADMIN_USER` (string - default user; must match secret)
* `JENKINS_TOKEN` (string - token set for default user; must match secret)
* `JENKINS_URL` (string - Jenkins URL)
* `jenkins-job-builder-github` (SSH Private Key with Username, with access to `JJB_REPO`)

---

### Fun Things

Configure a job (or multiple) with a Generic Webhook Trigger and trigger with curl:
```
curl --user admin:114d13730ad0d9294e4b424bf8d873c26d -X POST -L -H "Content-Type: application/json" "http://JENKINS_URL/generic-webhook-trigger/invoke?token=newtag" -d '{"job": "podtest"}'
```


Easily create an AWS Secret Manager secret
* https://plugins.jenkins.io/aws-secrets-manager-credentials-provider/
```
aws secretsmanager create-secret --name JENKINS_TOKEN --secret-string '114d13730ad0d9294e4b424bf8d873c26d' --tags 'Key=jenkins:credentials:type,Value=string' --description 'Jenkins API Token'
```
---

TODO: Update resource limits and concurrency for k8s agents
