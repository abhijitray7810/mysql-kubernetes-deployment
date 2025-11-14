# MySQL Kubernetes Quick Reference

A quick reference guide for common commands and operations.

## 🚀 Quick Deploy

```bash
# Deploy everything
kubectl apply -f mysql-deployment.yaml

# Deploy step-by-step
kubectl apply -f manifests/secrets.yaml
kubectl apply -f manifests/pv-pvc.yaml
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service.yaml
```

## 📋 Essential Commands

### Check Status

```bash
# All resources
kubectl get all -l app=mysql

# Specific resources
kubectl get secrets | grep mysql
kubectl get pv,pvc
kubectl get deployment mysql-deployment
kubectl get pods -l app=mysql
kubectl get svc mysql

# Detailed info
kubectl describe deployment mysql-deployment
kubectl describe pod <pod-name>
kubectl describe svc mysql
```

### View Logs

```bash
# Current logs
kubectl logs deployment/mysql-deployment

# Follow logs
kubectl logs -f deployment/mysql-deployment

# Last 50 lines
kubectl logs deployment/mysql-deployment --tail=50

# Previous logs (if crashed)
kubectl logs deployment/mysql-deployment --previous
```

### Connect to MySQL

```bash
# Via NodePort (external)
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
mysql -h $NODE_IP -P 30007 -u kodekloud_joy -pGyQkFRVNr3 kodekloud_db10

# Via port-forward (local)
kubectl port-forward svc/mysql 3306:3306
mysql -h 127.0.0.1 -P 3306 -u kodekloud_joy -pGyQkFRVNr3 kodekloud_db10

# Inside cluster (temporary pod)
kubectl run mysql-client --rm -it --image=mysql:5.7 --restart=Never -- \
  mysql -h mysql -u kodekloud_joy -pGyQkFRVNr3 kodekloud_db10

# Direct pod access
kubectl exec -it deployment/mysql-deployment -- \
  mysql -u kodekloud_joy -pGyQkFRVNr3 kodekloud_db10
```

## 🔧 Maintenance

### Update Operations

```bash
# Restart deployment
kubectl rollout restart deployment/mysql-deployment

# Check rollout status
kubectl rollout status deployment/mysql-deployment

# Update image
kubectl set image deployment/mysql-deployment mysql=mysql:8.0

# Rollback
kubectl rollout undo deployment/mysql-deployment

# View rollout history
kubectl rollout history deployment/mysql-deployment
```

### Scale (Not recommended for MySQL)

```bash
# Scale up/down (keep at 1 for MySQL)
kubectl scale deployment mysql-deployment --replicas=1
```

### Secret Management

```bash
# View secrets (base64 encoded)
kubectl get secret mysql-root-pass -o yaml

# Decode secret
kubectl get secret mysql-root-pass -o jsonpath='{.data.password}' | base64 -d

# Update secret
kubectl delete secret mysql-root-pass
kubectl create secret generic mysql-root-pass --from-literal=password=NewPass123
kubectl rollout restart deployment/mysql-deployment

# Edit secret directly
kubectl edit secret mysql-root-pass
```

## 💾 Backup & Restore

### Backup

```bash
# Backup single database
kubectl exec deployment/mysql-deployment -- \
  mysqldump -u root -pYUIidhb667 kodekloud_db10 > backup-$(date +%Y%m%d).sql

# Backup all databases
kubectl exec deployment/mysql-deployment -- \
  mysqldump -u root -pYUIidhb667 --all-databases > full-backup-$(date +%Y%m%d).sql

# Copy files from pod
kubectl cp mysql-deployment-<pod-id>:/var/lib/mysql ./mysql-data-backup
```

### Restore

```bash
# Restore from backup
kubectl exec -i deployment/mysql-deployment -- \
  mysql -u root -pYUIidhb667 kodekloud_db10 < backup.sql

# Copy files to pod
kubectl cp ./mysql-data-backup mysql-deployment-<pod-id>:/var/lib/mysql
```

## 🔍 Troubleshooting

### Quick Diagnostics

```bash
# Check pod status
kubectl get pods -l app=mysql -o wide

# View events
kubectl get events --sort-by='.lastTimestamp' | grep mysql

# Describe resources
kubectl describe pod <pod-name>
kubectl describe pvc mysql-pv-claim
kubectl describe svc mysql

# Check endpoints
kubectl get endpoints mysql

# Resource usage
kubectl top pod -l app=mysql
kubectl top node
```

### Common Issues

```bash
# Pod not starting - check logs
kubectl logs <pod-name>
kubectl describe pod <pod-name>

# PVC not binding - check PV
kubectl get pv
kubectl describe pvc mysql-pv-claim

# Connection refused - check service
kubectl get svc mysql
kubectl get endpoints mysql

# Secret not found - verify secrets
kubectl get secrets | grep mysql

# Permission denied - check RBAC
kubectl auth can-i create deployments
kubectl auth can-i get secrets
```

### Debug Pod

```bash
# Run debug container
kubectl run debug --rm -it --image=busybox --restart=Never -- sh

# Inside debug container:
nc -zv mysql 3306
nslookup mysql
ping mysql
```

## 🧹 Cleanup

### Partial Cleanup

```bash
# Delete deployment only
kubectl delete deployment mysql-deployment

# Delete service only
kubectl delete svc mysql

# Delete secrets only
kubectl delete secret mysql-root-pass mysql-user-pass mysql-db-url

# Delete storage
kubectl delete pvc mysql-pv-claim
kubectl delete pv mysql-pv
```

### Complete Cleanup

```bash
# Delete everything
kubectl delete -f mysql-deployment.yaml

# Or delete by label
kubectl delete all,pvc,secret -l app=mysql

# Force delete stuck resources
kubectl delete pod <pod-name> --force --grace-period=0
```

## 📊 Monitoring

### Resource Usage

```bash
# Pod metrics
kubectl top pod -l app=mysql

# Node metrics
kubectl top nodes

# Describe for limits/requests
kubectl describe pod <pod-name> | grep -A 5 "Limits\|Requests"
```

### MySQL Queries

```bash
# Check processes
kubectl exec deployment/mysql-deployment -- \
  mysql -u root -pYUIidhb667 -e "SHOW PROCESSLIST;"

# Check status
kubectl exec deployment/mysql-deployment -- \
  mysql -u root -pYUIidhb667 -e "SHOW STATUS;"

# Check variables
kubectl exec deployment/mysql-deployment -- \
  mysql -u root -pYUIidhb667 -e "SHOW VARIABLES LIKE '%version%';"

# Check databases
kubectl exec deployment/mysql-deployment -- \
  mysql -u root -pYUIidhb667 -e "SHOW DATABASES;"
```

## 🔐 Security

### Check Secrets

```bash
# List all secrets
kubectl get secrets

# Verify secret data
kubectl get secret mysql-root-pass -o jsonpath='{.data}' | jq

# Check environment variables in pod
kubectl exec deployment/mysql-deployment -- env | grep MYSQL
```

### Network Testing

```bash
# Test internal connectivity
kubectl run test --rm -it --image=busybox --restart=Never -- \
  nc -zv mysql 3306

# Test external connectivity (NodePort)
curl -v telnet://NODE_IP:30007
```

## 📝 Common MySQL Commands

```sql
-- Show databases
SHOW DATABASES;

-- Use database
USE kodekloud_db10;

-- Show tables
SHOW TABLES;

-- Show users
SELECT User, Host FROM mysql.user;

-- Create table
CREATE TABLE test (id INT PRIMARY KEY, name VARCHAR(50));

-- Insert data
INSERT INTO test VALUES (1, 'Test');

-- Query data
SELECT * FROM test;

-- Drop table
DROP TABLE test;

-- Show grants
SHOW GRANTS FOR 'kodekloud_joy'@'%';
```

## ⚡ One-Liners

```bash
# Get pod name
POD=$(kubectl get pod -l app=mysql -o jsonpath='{.items[0].metadata.name}')

# Quick status check
kubectl get all,pvc,secret -l app=mysql

# Follow logs of latest pod
kubectl logs -f $(kubectl get pod -l app=mysql -o jsonpath='{.items[0].metadata.name}')

# Execute MySQL command
kubectl exec deployment/mysql-deployment -- mysql -u root -pYUIidhb667 -e "SHOW DATABASES;"

# Port forward in background
kubectl port-forward svc/mysql 3306:3306 &

# Check if MySQL is ready
kubectl exec deployment/mysql-deployment -- mysqladmin ping -h localhost

# Get all resource usage
kubectl top pod -l app=mysql && kubectl top node
```

## 📦 Export/Import

### Export Configuration

```bash
# Export all resources
kubectl get deployment mysql-deployment -o yaml > deployment-backup.yaml
kubectl get svc mysql -o yaml > service-backup.yaml
kubectl get pvc mysql-pv-claim -o yaml > pvc-backup.yaml

# Export secrets (for backup only - keep secure!)
kubectl get secret mysql-root-pass -o yaml > secrets-backup.yaml
```

### Import to Another Cluster

```bash
# Set context to new cluster
kubectl config use-context new-cluster

# Apply resources
kubectl apply -f secrets-backup.yaml
kubectl apply -f pvc-backup.yaml
kubectl apply -f deployment-backup.yaml
kubectl apply -f service-backup.yaml
```

## 🎯 Useful Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
# General kubectl
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'

# MySQL specific
alias kmysql='kubectl exec -it deployment/mysql-deployment -- mysql -u root -pYUIidhb667'
alias kmysql-logs='kubectl logs -f deployment/mysql-deployment'
alias kmysql-status='kubectl get all,pvc,secret -l app=mysql'
alias kmysql-connect='kubectl port-forward svc/mysql 3306:3306'
```

## 🆘 Emergency Procedures

### Pod Crashed

```bash
# 1. Check logs
kubectl logs <pod-name> --previous

# 2. Describe pod
kubectl describe pod <pod-name>

# 3. Check events
kubectl get events --sort-by='.lastTimestamp'

# 4. Restart deployment
kubectl rollout restart deployment/mysql-deployment
```

### Data Corruption

```bash
# 1. Backup current state
kubectl exec deployment/mysql-deployment -- \
  mysqldump -u root -pYUIidhb667 --all-databases > emergency-backup.sql

# 2. Check MySQL logs
kubectl logs deployment/mysql-deployment | grep -i error

# 3. Run MySQL repair
kubectl exec deployment/mysql-deployment -- \
  mysqlcheck -u root -pYUIidhb667 --auto-repair --all-databases

# 4. Restore from backup if needed
kubectl exec -i deployment/mysql-deployment -- \
  mysql -u root -pYUIidhb667 < good-backup.sql
```

### Storage Full

```bash
# 1. Check disk usage
kubectl exec deployment/mysql-deployment -- df -h

# 2. Check MySQL data size
kubectl exec deployment/mysql-deployment -- \
  du -sh /var/lib/mysql

# 3. Clean up MySQL logs
kubectl exec deployment/mysql-deployment -- \
  mysql -u root -pYUIidhb667 -e "PURGE BINARY LOGS BEFORE NOW();"

# 4. If needed, increase PVC size (if storage class supports it)
kubectl patch pvc mysql-pv-claim -p '{"spec":{"resources":{"requests":{"storage":"500Mi"}}}}'
```

---

**Pro Tip**: Bookmark this page for quick reference during operations!

**Last Updated**: November 2024
