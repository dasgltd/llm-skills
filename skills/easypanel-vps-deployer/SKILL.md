---
name: easypanel-vps-deployer
description: Use this skill whenever the user asks to deploy a new Docker container, host a new project on their VPS, or alter/monitor apps managed by EasyPanel. Trigger this when mentions of VPS deployment, EasyPanel, .env modifications on the server, or pushing code to the remote server occur.
---

# EasyPanel & VPS Deployer

This skill enables full, autonomous management and deployment of applications to the user's VPS (which runs EasyPanel/Docker), without requiring manual intervention or asking for credentials.

## 1. Credentials & Connection Details
Do not ask the user for passwords. You should authenticate using the SSH key already configured on the user's Mac.
- **Host/IP**: `<YOUR_VPS_IP>` (or the alias configured in `~/.ssh/config`)
- **User**: `root`
- **Authentication**: SSH Key (Assumed to be configured by the user and authorized on the server)

**Important**: Never use passwords or `expect` scripts. Rely entirely on native SSH key authentication.

## 2. Typical Workflows

### A. Deploying / Uploading Files (Rsync)
When you need to upload a local project or update files on the server, use standard `rsync` with SSH key authentication:
```bash
rsync -avz --delete "local-dir/" "root@<YOUR_VPS_IP>:/path/on/server/"
```
### B. Managing .env Files
If a project needs environment variables updated, create or modify the `.env` file directly on the VPS via SSH:
```bash
# Example SSH command to update .env
ssh root@<YOUR_VPS_IP> "echo 'KEY=VALUE' >> /path/to/.env"
```

### C. Docker & EasyPanel Management
EasyPanel manages apps via Docker. To monitor or alter a service:
- List containers: `docker ps`
- Restart an app: `docker restart <container_id_or_name>`
- View logs: `docker logs --tail 100 <container>`
- Execute commands inside a container: `docker exec -i <container> bash -c '...'`

## 3. Execution Rules
1. **Zero User Intervention**: Execute the deployment entirely in the background. Use the `run_command` tool to run your standard `ssh` or `rsync` commands.
2. **Verify Success**: After deploying or modifying a container, run a status check (e.g., `ssh root@<YOUR_VPS_IP> docker ps` or `curl`) to ensure the service is up and running before telling the user the task is complete.
