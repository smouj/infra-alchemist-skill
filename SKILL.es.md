name: Infrastructure Alchemist
slug: infra-alchemist
description: Transmuta configuraciones complejas de infraestructura en elegancia operativa pura a través de flujos de trabajo automatizados y auto-reparables
author: Kilo-Core
version: 1.0.0
tags:
  - cloud
  - terraform
  - kubernetes
  - automation
  - self-healing
requires:
  - terraform >= 1.0
  - kubectl
  - awscli OR gcloud OR az
  - jq
  - yq
  - helm (optional)
---

## Propósito

Infrastructure Alchemist convierte definiciones de infraestructura declarativas en despliegues confiables y listos para producción con redes de seguridad integradas y capacidades de auto-reparación.

**Casos de uso reales:**
- Desplegar infraestructura multi-región AWS/GCP/Azure con terraform y failover automatizado
- Implementar flujos de trabajo GitOps usando ArgoCD/Flux con detección de deriva automatizada y reconciliación
- Crear clústeres Kubernetes auto-reparables que reinician automáticamente pods fallidos, reemplazan nodos no saludables y escalan basándose en métricas personalizadas
- Construir pipelines automatizados de recuperación ante desastres que hacen snapshots de bases de datos, respaldan etcd y replican configuraciones entre regiones
- Optimizar costos en la nube mediante redimensionamiento automatizado de recursos basado en patrones de uso
- Migrar infraestructura heredada a IaC moderno con importación de estado y validación
- Gestionar inyección de secretos compleja usando HashiCorp Vault o KMS en la nube
- Configurar organizaciones multi-cuenta AWS con SCPs, roles cross-account y logging centralizado

## Alcance

**Categorías de comandos soportadas:**

### Operaciones Terraform
- `terraform init -backend-config="bucket=my-bucket" -reconfigure`
- `terraform plan -var-file=prod.tfvars -out=tfplan -detailed-exitcode`
- `terraform apply tfplan -auto-approve`
- `terraform destroy -var-file=prod.tfvars -auto-approve`
- `terraform state pull > backup-$(date +%Y%m%d-%H%M%S).tfstate`
- `terraform workspace new staging && terraform workspace select staging`
- `terraform validate -json`
- `terraform fmt -check -recursive`
- `terraform taint aws_instance.web_server`
- `terraform import aws_instance.web_server i-1234567890abcdef0`

### Operaciones Kubernetes
- `kubectl apply -f manifests/ --namespace=production --server-side --force-conflicts`
- `kubectl rollout status deployment/app-name -w --timeout=5m`
- `kubectl set image deployment/app-name container=image:tag --record`
- `kubectl scale deployment/app-name --replicas=5`
- `kubectl autoscale deployment/app-name --cpu-percent=70 --min=2 --max=10`
- `kubectl delete pod --field-selector=status.phase=Failed -n default`
- `kubectl get events --sort-by='.lastTimestamp' -n production --field-selector=type!=Normal`
- `kubectl cordon node-name && kubectl drain node-name --ignore-daemonsets --delete-emptydir-data`
- `kubectl config use-context staging-cluster`
- `kubectl create secret generic db-creds --from-literal=password=$(aws secretsmanager get-secret-value --secret-id db/password --query SecretString --output text)`

### CLI de Proveedor en la Nube
**AWS:**
- `aws s3 sync infrastructure-states/ s3://tf-state-bucket/production/ --delete`
- `aws ec2 describe-instances --filters "Name=tag:Environment,Values=production" --query "Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress]" --output table`
- `aws autoscaling set-instance-protection --instance-ids i-12345 --no-protected-from-scale-in`
- `aws rds create-db-snapshot --db-instance-identifier production-db --db-snapshot-identifier backup-$(date +%Y%m%d)`
- `aws cloudformation deploy --template-file template.yaml --stack-name production-stack --parameter-overrides Environment=production --capabilities CAPABILITY_NAMED_IAM`

**GCP:**
- `gcloud compute instances list --filter="labels.env:production" --format="table(name, status, externalIP)"`
- `gcloud container clusters get-credentials production-cluster --region us-central1`
- `gcloud sql backups create --instance=production-db`
- `gcloud deployment-manager deployments create production --config infrastructure.yaml`

**Azure:**
- `az vm list --resource-group production-rg --show-details --query "[].{name:name, status:powerState, ip:privateIps}" -o table`
- `az aks get-credentials --resource-group production-rg --name production-cluster --admin`
- `az backup protection enable-for-vm --resource-group production-rg --vm-name web-01 --vault-name backup-vault --policy-name DefaultPolicy`

### Operaciones Helm
- `helm upgrade --install production-app ./chart -n production --create-namespace --set image.tag=${TAG} --set replicaCount=3 --wait --timeout 5m`
- `helm rollback production-app 2`
- `helm get values production-app -n production --all`
- `helm test production-app -n production --logs`
- `helm history production-app -n production`

### Automatización Auto-Reparable
- `kubectl get pods -n production --field-selector=status.phase!=Running -o json | jq -r '.items[] | select(.status.reason=="OOMKilled") | .metadata.name' | xargs -r kubectl delete pod -n production`
- `watch -n 30 "kubectl get pods -n production | grep -E '(CrashLoopBackOff|Error)' && kubectl rollout restart deployment -n production --all"`
- `curl -sf http://localhost:8080/health || (kubectl rollout restart deployment/monitoring -n monitoring && systemctl restart prometheus)`

## Proceso de Trabajo

1. **Análisis de Intención:** Analizar el prompt del usuario para identificar la plataforma objetivo (AWS/GCP/Azure/K8s), tipo de operación (desplegar/actualizar/destruir/reparar) y alcance (recurso único vs stack completo)
2. **Detección de Entorno:** Auto-detectar contexto desde KUBECONFIG, terraform workspace, AWS_PROFILE, GOOGLE_APPLICATION_CREDENTIALS; fallar rápido si falta
3. **Resolución de Dependencias:** Verificar binarios requeridos (terraform version >= 1.0, conectividad cluster kubectl, auth cloud CLI); instalar missing via package manager si configurado
4. **Validación Pre-Vuelo:**
   - Ejecutar `terraform validate` para cambios IaC
   - Verificar `kubectl cluster-info` para operaciones K8s
   - Revisar `aws sts get-caller-identity` para auth en la nube
   - Asegurar state bucket/backend accesible
   - Confirmar cuotas de nube suficientes
5. **Generación de Plan:** Crear plan de ejecución detallado con `terraform plan -out=tfplan` o `kubectl apply --dry-run=client -o yaml`; mostrar resumen legible con cuenta de recursos a cambiar
6. **Puntos de Seguridad:**
   - Detectar operaciones destroy; requerir flag explícito `--approve-destroy` si > 5 recursos
   - Revisar pérdida de datos potencial (RDS deletion protection deshabilitado, EBS volumes sin snapshot)
   - Verificar que replica count no baje del mínimo
   - Asegurar maintenance windows para entornos de producción
7. **Ejecución con Logging:** Ejecutar comandos con streaming de output en tiempo real; capturar stdout/stderr completo a `./logs/infra-alchemist-$(date +%Y%m%d-%H%M%S).log`
8. **Checks de Salud Post-Ejecución:**
   - Esperar a que pods K8s alcancen estado Ready (`kubectl wait --for=condition=Ready pod --all --timeout=180s`)
   - Verificar que health checks de load balancer estén passing
   - Ejecutar checks de integración si endpoints disponibles
   - Confirmar que terraform outputs coincidan con valores esperados
9. **Gestión de Estado:** Backup terraform state a S3/GCS/Azure Blob; commit plan files a git para auditoría; podar backups antiguos > 30 días
10. **Generación de Resumen:** Producir reporte legible con IDs de recursos, URLs de endpoints, ubicación de credenciales, instrucciones de rollback y ventana de siguiente mantenimiento

## Reglas de Oro

1. **Nunca saltar terraform plan** - Siempre revisar cambios antes de apply; plan debe almacenarse en git
2. **Backups de estado obligatorios** - Backup terraform state antes de cada apply; mantener 3 versiones más recientes
3. **No secrets en texto plano** - Nunca pasar passwords via `-var 'password=secret'`; usar vault o `-var-file` con permisos restringidos
4. **Radio de explosión mínimo** - Usar terraform workspaces, K8s namespaces y cloud tags para aislar entornos; limitar cambios en producción a single service cuando posible
5. **Siempre preservar deletion protection** - RDS, Redshift y otras bases de datos deben tener deletion protection habilitada en producción
6. **Requerir health checks** - Validación post-deployment requerida; si health check falla, automáticamente trigger rollback
7. **Principio de inmutabilidad** - Nunca modificar recursos existentes in-place; preferir patrones replace-with-no-downtime usando blue/green o rolling updates
8. **Etiquetar todo** - Todos los cloud resources deben tener tags: Environment, Owner, ManagedBy=infra-alchemist, GitCommit
9. **Asumir particiones de red** - Diseñar para fallos parciales; usar retry logic con exponential backoff en scripts personalizados
10. **Rollback antes de forward** - Para cualquier cambio en producción, verificar que el procedimiento de rollback funciona en staging primero

## Ejemplos

### Ejemplo 1: Desplegar Aplicación Web Multi-Nivel

**Prompt del usuario:** "Deploy production web app in AWS: VPC with public/private subnets, ECS cluster with ALB, RDS PostgreSQL, and ElastiCache Redis. Use production.tfvars."

**Comandos generados:**
```bash
# Authenticate
aws sso login --profile production

# Initialize terraform with remote backend
terraform init -backend-config="bucket=tf-state-production" -backend-config="dynamodb_table=tf-locks" -reconfigure

# Validate and format
terraform validate && terraform fmt -check -recursive
terraform plan -var-file=production.tfvars -out=tfplan-$(date +%Y%m%d-%H%M%S)

# Backup existing state if exists
if [ -f "terraform.tfstate" ]; then
  aws s3 cp terraform.tfstate s3://tf-state-backup/terraform.tfstate.$(date +%Y%m%d-%H%M%S)
fi

# Apply with monitoring
terraform apply tfplan-$(date +%Y%m%d-%H%M%S) -auto-approve

# Post-deployment verification
ALB_DNS=$(terraform output -raw alb_dns_name)
echo "Testing ALB health:"
curl -s -o /dev/null -w "%{http_code}" https://${ALB_DNS}/health || { echo "Health check failed!"; terraform apply -var-file=production.tfvars -auto-approve -refresh-only; exit 1; }

# Wait for ECS tasks to stabilize
aws ecs wait services-stable --cluster production-ecs --services web-app

# Backup new state
aws s3 sync . s3://tf-state-production/ --exclude="*.tfstate" --include="terraform.tfstate*"
```

**Output esperado:**
```
 terraform plan -var-file=production.tfvars

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.main will be created
  + resource "aws_vpc" "main" {
      + cidr_block           = "10.0.0.0/16"
      + id                   = (known after apply)
    }

  # aws_lb.main will be created
  + resource "aws_lb" "main" {
      + dns_name             = (known after apply)
      + arn                  = (known after apply)
    }

  # aws_db_instance.main will be created
  + resource "aws_db_instance" "main" {
      + identifier           = "production-db"
      + engine               = "postgres"
      + engine_version       = "15.4"
      + instance_class       = "db.t3.medium"
      + allocated_storage    = 100
      + username             = "admin"
      + password             = (sensitive value)
      + skip_final_snapshot  = false
      + deletion_protection  = true
    }

Plan: 23 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + alb_dns_name = (known after apply)

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't specify an "-out" parameter to save this plan. Terraform will always execute plans immediately after planning, even if you later decide to apply.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_vpc.main: Creating...
aws_vpc.main: Creation complete after 4s [id=vpc-0a1b2c3d4e5f6g7h]
aws_subnet.public[0]: Creating...
aws_subnet.public[1]: Creating...
...
...

Apply complete! Resources: 23 added, 0 changed, 0 destroyed.

Outputs:

alb_dns_name = "production-alb-123456.us-east-1.elb.amazonaws.com"
rds_endpoint = "production-db.123456789012.us-east-1.rds.amazonaws.com"
redis_endpoint = "production-cache.12345.cache.amazonaws.com"
```

### Ejemplo 2: Namespace Kubernetes Auto-Reparable

**Prompt del usuario:** "Heal the production namespace: restart all CrashLoopBackOff pods, cordon and drain nodes with memory pressure, and verify all deployments have min 2 replicas."

**Comandos generados:**
```bash
#!/bin/bash
NAMESPACE=production
ALERT_THRESHOLD=2

# 1. Find and delete CrashLoopBackOff pods
echo "Checking for CrashLoopBackOff pods..."
FAILED_PODS=$(kubectl get pods -n $NAMESPACE -o json | jq -r '.items[] | select(.status.containerStatuses[0].state.waiting.reason=="CrashLoopBackOff") | .metadata.name')
if [ -n "$FAILED_PODS" ]; then
  echo "Deleting failed pods:"
  echo "$FAILED_PODS"
  echo "$FAILED_PODS" | xargs -r kubectl delete pod -n $NAMESPACE
else
  echo "No CrashLoopBackOff pods found."
fi

# 2. Check for nodes with memory pressure
echo -e "\nChecking node health..."
MEMORY_PRESSURE_NODES=$(kubectl get nodes -o json | jq -r '.items[] | select(.status.conditions[] | select(.reason=="MemoryPressure" and .status=="True")) | .metadata.name')
if [ -n "$MEMORY_PRESSURE_NODES" ]; then
  echo "Nodes with memory pressure: $MEMORY_PRESSURE_NODES"
  for NODE in $MEMORY_PRESSURE_NODES; do
    echo "Cordoning $NODE and draining pods..."
    kubectl cordon $NODE
    kubectl drain $NODE --ignore-daemonsets --delete-emptydir-data --force
  done
else
  echo "All nodes healthy."
fi

# 3. Verify replica counts
echo -e "\nVerifying deployment replicas..."
UNDER_REPLICATED=$(kubectl get deployments -n $NAMESPACE -o json | jq -r '.items[] | select(.spec.replicas < '"$ALERT_THRESHOLD"') | .metadata.name')
if [ -n "$UNDER_REPLICATED" ]; then
  echo "WARNING: Deployments with less than $ALERT_THRESHOLD replicas:"
  echo "$UNDER_REPLICATED"
  echo "Scaling to minimum..."
  echo "$UNDER_REPLICATED" | xargs -r -I {} kubectl scale deployment {} --replicas=$ALERT_THRESHOLD -n $NAMESPACE
else
  echo "All deployments meet minimum replica count."
fi

# 4. Check pod distribution across nodes
echo -e "\nChecking pod distribution (anti-affinity)..."
IMBALANCED_NODES=$(kubectl get pods -n $NAMESPACE -o json | jq -r '.items[].spec.nodeName' | sort | uniq -c | awk '$1 > 15 {print $2}')
if [ -n "$IMBALANCED_NODES" ]; then
  echo "Nodes with high pod count: $IMBALANCED_NODES (consider rebalancing)"
fi

# 5. Final health status
echo -e "\n=== FINAL HEALTH STATUS ==="
kubectl get pods -n $NAMESPACE
kubectl get nodes
kubectl get hpa -n $NAMESPACE

echo "Self-healing complete. Monitor logs for 5 minutes."
```

**Output esperado:**
```
Checking for CrashLoopBackOff pods...
Deleting failed pods:
payment-service-6d4b5f6c9b-2kxj8
user-api-7c8d9e0f1g-5hjk9
No CrashLoopBackOff pods found.

Checking node health...
Nodes with memory pressure: ip-10-0-1-15.ec2.internal ip-10-0-1-22.ec2.internal
Cordoning ip-10-0-1-15.ec2.internal and draining pods...
node/ip-10-0-1-15.ec2.internal cordoned
WARNING: ignoring DaemonSet pods because --ignore-daemonsets=true; deleting pod payment-service-6d4b5f6c9b-2kxj8
...
pod/payment-service-6d4b5f6c9b-2kxj8 deleted
node/ip-10-0-1-15.ec2.internal drained

Verifying deployment replicas...
All deployments meet minimum replica count.

=== FINAL HEALTH STATUS ===
NAME                         READY   STATUS    RESTARTS   AGE
payment-service-6d4b5f6c9b  2/2     Running   0          30s
user-api-7c8d9e0f1g         2/2     Running   0          30s
...

Self-healing complete. Monitor logs for 5 minutes.
```

### Ejemplo 3: Detección y Remediación de Deriva en Terraform

**Prompt del usuario:** "Detect and report any drift in the production terraform state. Auto-remediate if safe, otherwise flag for manual review."

**Comandos generados:**
```bash
#!/bin/bash
WORKSPACE=production
DRIFT_REPORT="/tmp/drift-report-$(date +%Y%m%d-%H%M%S).txt"
BACKUP_DIR="/tmp/tf-state-backup/$(date +%Y%m%d-%H%M%S)"

mkdir -p $BACKUP_DIR

# 1. Select workspace
terraform workspace select $WORKSPACE

# 2. Refresh state (detect drift)
echo "Refreshing terraform state to detect drift..."
terraform refresh -var-file=production.tfvars > /dev/null 2>&1

# 3. Generate drift report
echo "=== DRIFT DETECTION REPORT ===" > $DRIFT_REPORT
echo "Workspace: $WORKSPACE" >> $DRIFT_REPORT
echo "Timestamp: $(date -Iseconds)" >> $DRIFT_REPORT
echo "" >> $DRIFT_REPORT

# Compare current state with planned state
terraform plan -var-file=production.tfvars -detailed-exitcode > /tmp/plan-output.txt 2>&1
EXIT_CODE=$?

if [ $EXIT_CODE -eq 0 ]; then
  echo "STATUS: No drift detected. Infrastructure matches Terraform configuration." >> $DRIFT_REPORT
  cat $DRIFT_REPORT
  exit 0
elif [ $EXIT_CODE -eq 2 ]; then
  echo "STATUS: Drift detected. Changes found outside of Terraform control." >> $DRIFT_REPORT
  echo "" >> $DRIFT_REPORT
  echo "DRIFTED RESOURCES:" >> $DRIFT_REPORT
  grep -E "^  # |^  -|^  +" /tmp/plan-output.txt | head -50 >> $DRIFT_REPORT
else
  echo "ERROR: Terraform plan failed with exit code $EXIT_CODE" >> $DRIFT_REPORT
  cat $DRIFT_REPORT
  exit 1
fi

# 4. Determine if auto-remediation is safe
AUTO_REMEDIATE=false
if grep -q "aws_s3_bucket_versioning" /tmp/plan-output.txt && \
   grep -q "aws_instance" /tmp/plan-output.txt; then
  echo "WARNING: Mixed resource changes detected. Manual review required." >> $DRIFT_REPORT
else
  AUTO_REMEDIATE=true
  echo "INFO: Safe to auto-remediate (only simple resource updates detected)." >> $DRIFT_REPORT
fi

# 5. Backup current state before any changes
echo "Backing up terraform state..."
terraform state pull > $BACKUP_DIR/terraform.tfstate

# 6. Auto-remediate if safe
if [ "$AUTO_REMEDIATE" = true ]; then
  echo "Applying terraform configuration to remediate drift..."
  terraform apply -var-file=production.tfvars -auto-approve
  
  # Verify remediation
  terraform plan -var-file=production.tfvars -detailed-exitcode > /tmp/verify-plan.txt 2>&1
  if [ $? -eq 0 ]; then
    echo "SUCCESS: Drift remediated. Infrastructure now matches configuration." >> $DRIFT_REPORT
    # Sync state to remote backend
    aws s3 sync . s3://tf-state-production/production/ --exclude="*.tfstate" --include="terraform.tfstate*"
  else
    echo "ERROR: Remediation failed or introduced new drift." >> $DRIFT_REPORT
    exit 1
  fi
else
  echo "Manual intervention required. Run: terraform apply -var-file=production.tfvars after review." >> $DRIFT_REPORT
fi

cat $DRIFT_REPORT
```

**Output esperado:**
```
=== DRIFT DETECTION REPORT ===
Workspace: production
Timestamp: 2026-03-01T14:23:45+00:00

STATUS: Drift detected. Changes found outside of Terraform control.

DRIFTED RESOURCES:
  # aws_s3_bucket.logging will be updated in-place
  ~ resource "aws_s3_bucket" "logging" {
      id                    = "production-logs-bucket"
      # (3 unchanged attributes hidden)

      ~ versioning {
          # (1 unchanged attribute hidden)
            enabled    = false -> true
        }
    }

  # aws_instance.web_server[0] will be updated in-place
  ~ resource "aws_instance" "web_server" {
      id                    = "i-0a1b2c3d4e5f6g7h"
      # (2 unchanged attributes hidden)

      ~ instance_type        = "t3.medium" -> "t3.large"
        # (1 unchanged attribute hidden)
    }

INFO: Safe to auto-remediate (only simple resource updates detected).
Backing up terraform state...
Saved state to /tmp/tf-state-backup/20260301-142345/terraform.tfstate

Applying terraform configuration to remediate drift...
aws_s3_bucket.logging: Modifying...
aws_s3_bucket.logging: Modifications complete after 2s [id=production-logs-bucket]
aws_instance.web_server[0]: Modifying...
aws_instance.web_server[0]: Modifications complete after 15s [id=i-0a1b2c3d4e5f6g7h]

Apply complete! Resources: 0 added, 2 changed, 0 destroyed.

SUCCESS: Drift remediated. Infrastructure now matches configuration.
```

## Comandos de Rollback

### Rollback Terraform
```bash
# List terraform state backups
aws s3 ls s3://tf-state-backup/ --recursive | grep terraform.tfstate

# Restore previous state
LATEST_BACKUP=$(aws s3 ls s3://tf-state-backup/ --recursive | grep terraform.tfstate | tail -1 | awk '{print $4}')
aws s3 cp s3://tf-state-backup/${LATEST_BACKUP} terraform.tfstate.backup

# Switch workspace if needed
terraform workspace select production

# Revert to backup state (dry-run first)
terraform apply -var-file=production.tfvars terraform.tfstate.backup -refresh-only -detailed-exitcode

# Force restore if needed
terraform state replace-provider "hashicorp/aws" "hashicorp/aws" -auto-approve
terraform apply -var-file=production.tfvars -refresh-only
```

### Rollback Kubernetes
```bash
# Rollback specific deployment to previous revision
kubectl rollout undo deployment/app-name --to-revision=3

# Check rollout history
kubectl rollout history deployment/app-name

# Pause and resume to control rollback speed
kubectl rollout pause deployment/app-name
kubectl rollout resume deployment/app-name

# Rollback all deployments in namespace (emergency)
kubectl get deployments -n production -o json | jq -r '.items[].metadata.name' | xargs -r -I {} kubectl rollout undo deployment/{} -n production

# Restore from ConfigMap backup
kubectl create configmap app-config-backup --from-file=backup-config.yaml -n production
kubectl rollout restart deployment/app-name -n production --field-manager=rollback-$(date +%s)
```

### Rollbacks Específicos por Proveedor en la Nube

**AWS:**
```bash
# Restore RDS from latest automated snapshot (within retention period)
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier production-db \
  --target-db-instance-identifier production-db-restored-$(date +%Y%m%d) \
  --restore-time "$(date -d '5 minutes ago' -Iseconds)" \
  --db-instance-class db.t3.medium

# Update Route53 to point to previous ELB if ALB changed
aws route53 change-resource-record-sets --hosted-zone-id Z1234567890ABC \
  --change-batch file://rollback-alb.json

# Undo CloudFormation stack update (if update failed)
aws cloudformation continue-update-rollback --stack-name production-stack

# Revert S3 bucket changes
aws s3api put-bucket-versioning --bucket production-logs-bucket \
  --versioning-configuration Status=Enabled
```

**GCP:**
```bash
# Restore GKE cluster from backup (if backup enabled)
gcloud container clusters restore-production \
  --backup-file gs://backup-bucket/gke-backup-$(date +%Y%m%d).yaml

# Roll back Deployment Manager deployment to previous
gcloud deployment-manager deployments rollback production --revision=5

# Restore Cloud SQL from backup
gcloud sql instances restore-backup production-db \
  --backup-instance=production-db \
  --backup-time="2026-03-01T14:00:00Z"

# Revert instance template and recreate MIG
gcloud compute instance-templates create rollback-template-$(date +%Y%m%d) \
  --source-instance-template=production-template-$(date -d '1 hour ago' +%Y%m%d)
gcloud compute instance-groups managed rolling-action start-update production-mig \
  --version=template=rollback-template-$(date +%Y%m%d)
```

**Azure:**
```bash
# Restore VM from recovery services vault
az backup recoverypoint show --vault-name backup-vault \
  --container-name "iaasvmcontainer;iaasvmcontainerv2;production-rg;web-01" \
  --item-name "vm;iaasvmcontainerv2;production-rg;web-01" \
  --query "properties.recoveryPointTime"
az backup restore restore-disks --vault-name backup-vault \
  --container-name "iaasvmcontainer;iaasvmcontainerv2;production-rg;web-01" \
  --item-name "vm;iaasvmcontainerv2;production-rg;web-01" \
  --recovery-point "2026-03-01T14:00:00Z" \
  --storage-account restorestorage

# Rollback AKS to previous GitOps commit (if using Flux)
kubectl -n flux get kustomization.kustomize.toolkit.fluxcd.io | grep production
# Edit Kustomization to point to previous commit SHA
kubectl edit kustomization production -n flux
# Force reconciliation
kubectl annotate kustomization production fluxcd.io/skip-note="trigger-reconcile" -n flux

# Redeploy ARM template to previous version
az deployment group create --resource-group production-rg \
  --template-file infrastructure-rollback.json \
  --parameters @production.parameters.json
```

### Rollback Global de Emergencia (Todos los Servicios)
```bash
#!/bin/bash
# Execute in order: databases first, then stateful apps, then stateless

echo "INITIATING EMERGENCY ROLLBACK - Confirm this action will cause downtime. Continue? [yes/NO]"
read CONFIRM
if [ "$CONFIRM" != "yes" ]; then exit 1; fi

# 0. Notify team
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"EMERGENCY ROLLBACK initiated by infra-alchemist at $(date)"}' \
  $SLACK_WEBHOOK_URL

# 1. Put all K8s deployments into maintenance mode
kubectl apply -f维护模式设置.yaml -n production

# 2. Restore databases from backups (AWS example)
for DB in production-db analytics-db; do
  SNAPSHOT=$(aws rds describe-db-snapshots --db-instance-identifier $DB \
    --query "reverse(sort_by(DBSnapshots, &SnapshotCreateTime))[0].DBSnapshotIdentifier" --output text)
  aws rds restore-db-instance-from-db-snapshot \
    --db-instance-identifier $DB-rollback-$(date +%Y%m%d) \
    --db-snapshot-identifier $SNAPSHOT \
    --db-instance-class db.t3.large
done

# 3. Restore terraform state from known good backup
GOOD_STATE=$(aws s3 ls s3://tf-state-backup/ --recursive | grep "terraform.tfstate" | grep "$(date -d '1 hour ago' +%Y%m%d)" | tail -1 | awk '{print $4}')
aws s3 cp s3://tf-state-backup/${GOOD_STATE} terraform.tfstate
terraform workspace select production
terraform apply -refresh-only -var-file=production.tfvars

# 4. Verify all systems down before proceeding
echo "All systems should be in maintenance mode. Press Enter to begin restore..."
read

# 5. Bring up databases first, wait for health
for DB in production-db analytics-db; do
  aws rds wait db-instance-available --db-instance-identifier $DB-rollback-$(date +%Y%m%d)
  # Update connection strings in parameter store
  ENDPOINT=$(aws rds describe-db-instances --db-instance-identifier $DB-rollback-$(date +%Y%m%d) --query "DBInstances[0].Endpoint.Address" --output text)
  aws ssm put-parameter --name "/production/${DB}/endpoint" --value $ENDPOINT --type String --overwrite
done

# 6. Redeploy all K8s applications
kubectl apply -f manifests/ -n production --server-side
kubectl wait --for=condition=ready pod --all -n production --timeout=600s

# 7. Verify health
for SERVICE in api web worker; do
  curl -s -f -o /dev/null -w "%{http_code}" https://$SERVICE.production.example.com/health || \
    { echo "Health check failed for $SERVICE"; exit 1; }
done

# 8. Announce rollback complete
curl -X POST -H 'Content-type: application/json' \
  --data "{\"text\":\"EMERGENCY ROLLBACK completed successfully. All services operational.\"}" \
  $SLACK_WEBHOOK_URL

echo "Rollback complete. Monitoring for 30 minutes before declaring victory."
```

## Dependencias y Requisitos

**Binarios del sistema:**
- terraform (v1.0+)
- kubectl (v1.24+)
- awscli OR gcloud OR az (al menos un proveedor en la nube)
- jq (v1.6+)
- yq (v4.30+)
- helm (v3.10+, opcional)
- git
- curl
- bash (v4.0+)

**Variables de entorno:**
```bash
# Cloud authentication
export AWS_PROFILE=production
export AWS_REGION=us-east-1
export GOOGLE_APPLICATION_CREDENTIALS=~/.config/gcloud/production-key.json
export AZURE_SUBSCRIPTION_ID=xxxx
export ARM_USE_MSI=true

# Terraform
export TF_IN_AUTOMATION=true
export TF_INPUT=false
export TF_LOG=ERROR
export TF_DATA_DIR=/tmp/terraform.tfstate.d

# Kubernetes
export KUBECONFIG=~/.kube/production-config
export KUBECTL_PLUGINS_PATH=~/.kube/plugins

# Backup locations
export TF_STATE_BUCKET=tf-state-production
export BACKUP_S3_PREFIX=infra-backups
export DRIFT_REPORT_SLACK_WEBHOOK=https://hooks.slack.com/services/xxx

# Self-healing thresholds
export SELF_HEAL_CRASHLOOP_TIMEOUT=300
export SELF_HEAL_MIN_REPLICAS=2
export NODE_DRAIN_TIMEOUT=600

# Feature flags
export INFRA_ALCHEMIST_AUTO_REMEDIATE=true
export INFRA_ALCHEMIST_DRY_RUN=false
export INFRA_ALCHEMIST_REQUIRE_APPROVAL_DESTROY=true
```

**Configuración de backend Terraform (requerida):**
```hcl
terraform {
  backend "s3" {
    bucket         = "tf-state-production"
    key            = "production/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-locks"
    encrypt        = true
    versioning     = true
  }
}
```

## Pasos de Verificación

**Después del despliegue de infraestructura:**
```bash
# 1. Verify terraform state matches cloud
terraform plan -detailed-exitcode || echo "State drift detected!"

# 2. Check all resources are healthy
aws elb describe-target-health --target-group-arn $(terraform output -raw tg_arn) | jq -r '.TargetHealthDescriptions[].TargetHealth.State' | grep -v healthy && echo "Unhealthy targets!" || echo "All targets healthy"

# 3. Verify network connectivity
kubectl get svc -n ingress
kubectl exec -it deployment/test-connectivity -n monitoring -- curl -s -f https://api.production.example.com/health

# 4. Confirm secrets are accessible
kubectl get secret db-password -n production -o jsonpath='{.data.password}' | base64 -d > /dev/null && echo "Secrets decryptable"

# 5. Validate IAM permissions
aws iam get-user && echo "IAM credentials valid"

# 6. Test failover (if multiple AZs)
aws rds reboot-db-instance --db-instance-identifier production-db --force-failover
sleep 180
kubectl get pods -n production | grep -v Running && echo "Failover caused issues!" || echo "Failover successful"
```

**Verificación específica de Kubernetes:**
```bash
# Pod readiness
kubectl wait --for=condition=Ready pod --all -n production --timeout=300s

# Service endpoints populated
kubectl get endpoints -n production | awk '$3 == 0 {print "No endpoints for " $1}' | grep -v "No endpoints for"

# HPA functionality
kubectl get hpa -n production -o json | jq -r '.items[] | select(.status.currentReplicas < .spec.minReplicas) | .metadata.name' && echo "HPA scale below minimum!"

# Resource quotas not exceeded
kubectl describe resourcequota -n production | grep -A 5 "hard:"

# Network policies applied
kubectl get networkpolicy -n production | wc -l | grep -q '[1-9]' && echo "Network policies in place" || echo "WARNING: No network policies"
```

## Solución de Problemas

**Problemas Comunes y Soluciones:**

1. **Terraform state lock atascado:**
```bash
# Identify lock
terraform force-unlock $(aws dynamodb get-item --table-name tf-locks --key '{"ID":{"S":"terraform-state-lock"}}' --query 'Item.LockID.S' --output text)
# OR
terraform force-unlock <lock-id>
# Prevention: Always set lock_timeout in backend config
```

2. **Kubernetes API server unavailable:**
```bash
# Check kubeconfig context
kubectl config current-context
# Test connectivity
kubectl cluster-info
# If using EKS, refresh credentials
aws eks update-kubeconfig --name production-cluster --alias production
# If still failing, verify security groups allow 443 from current IP
```

3. **Cloud CLI authentication expired:**
```bash
# AWS SSO
aws sso login --profile production
# GCP
gcloud auth login
gcloud config set account service-account@project.iam.gserviceaccount.com
# Azure
az login --use-device-code
az account set --subscription "Production Subscription"
```

4. **Terraform plan shows unexpected changes (phantom diff):**
```bash
# Refresh state
terraform refresh -var-file=production.tfvars
# Check for computed fields that cannot be imported
terraform plan -var-file=production.tfvars -refresh-only
# Use lifecycle ignore_changes for ephemeral attributes
# Example: lifecycle { ignore_changes = [public_ip] }
```

5. **Helm release stuck in pending-upgrade:**
```bash
# Check helm history
helm history release-name -n production
# Delete failed release (orphaned resources remain)
helm delete release-name --no-hooks
# Or uninstall and reinstall preserving previous values
helm get values release-name -n production > values.yaml
helm uninstall release-name -n production
helm install release-name ./chart -n production -f values.yaml
```

6. **EKS node group not scaling:**
```bash
# Check CloudFormation stack events
aws cloudformation describe-stack-events --stack-name EKS-NodeGroup-production
# Verify IAM role trust relationship
aws iam get-role --role-name NodeInstanceRole
# Check cluster autoscaler logs
kubectl logs -n kube-system deployment/cluster-autoscaler
# Ensure node group has available capacity
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name eks-nodegroup-production
```

7. **RDS snapshot restore fails with insufficient storage:**
```bash
# Check current instance storage
aws rds describe-db-instances --db-instance-identifier production-db --query "DBInstances[0].AllocatedStorage"
# Restore to larger instance, then downsize after
aws rds restore-db-instance-from-db-snapshot --db-instance-identifier temp-restore --db-snapshot-identifier snap-xxxx --db-instance-class db.t3.large --allocated-storage 200
# Create final instance from temporary
aws rds create-db-instance --db-instance-identifier production-db --db-instance-class db.t3.medium --engine postgres --master-username admin --master-user-password $(aws ssm get-parameter --name /prod/db/password --query Parameter.Value --output text) --allocated-storage 100 --no-publicly-accessible
```

8. **Self-healing script itself causing instability:**
```bash
# Add rate limiting and circuit breaker
MAX_CONCURRENT_DELETES=10
FAILED_OPERATIONS_LOG="/var/log/infra-heal-errors.log"

# Use timeout and retry with backoff
timeout 300s kubectl rollout restart deployment -n production --all || \
  echo "$(date): Rollout timeout" >> $FAILED_OPERATIONS_LOG

# Implement exponential backoff
for i in {1..5}; do
  kubectl get pods -n production | grep -q CrashLoopBackOff && \
    sleep $((2**i)) && \
    kubectl delete pod -n production --field-selector=status.phase=Failed || break
done
```

9. **S3 bucket sync failures during state backup:**
```bash
# Check bucket policy and encryption
aws s3api get-bucket-policy --bucket tf-state-production
aws s3api get-bucket-encryption --bucket tf-state-production

# Use SSE-KMS if required
aws s3 sync . s3://tf-state-production/ --storage-class STANDARD_IA \
  --sse aws:kms --sse-kms-key-id alias/tf-state-key

# Resume partial sync
aws s3 sync . s3://tf-state-production/ --size-only --quiet
```

10. **Network partitions causing split-brain in stateful services:**
```bash
# Use leader election for critical jobs (K8s example)
kubectl create configmap leader-elector-config --from-literal=lock-key=/locks/$(hostname)

# Implement retry with jitter
RETRY_COUNT=0
until [ $RETRY_COUNT -ge 5 ]; do
  kubectl apply -f manifests/ && break
  sleep $((RANDOM % 5))
  RETRY_COUNT=$((RETRY_COUNT + 1))
done

# Guard against partial application
PODS_BEFORE=$(kubectl get pods -n production --no-headers | wc -l)
kubectl apply -f manifests/ -n production
PODS_AFTER=$(kubectl get pods -n production --no-headers | wc -l)
if [ $PODS_AFTER -ne $PODS_BEFORE ] && [ $((PODS_AFTER - PODS_BEFORE)) -lt 0 ]; then
  echo "Pod count decreased unexpectedly! Initiating rollback..."
  kubectl rollout undo deployment -n production --all
fi
```
```