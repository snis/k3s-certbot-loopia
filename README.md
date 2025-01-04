# K3s Certbot with Loopia DNS Challenge

Automate SSL certificate management in K3s using Certbot with DNS-01 challenge via Loopia API. This project helps you automatically obtain and renew wildcard certificates for domains managed by Loopia.

## Prerequisites

- K3s cluster (single node or multi-node)
- Domain registered with Loopia
- Loopia API credentials (obtain from https://customerzone.loopia.com/api)
- Valid email for Let's Encrypt notifications

## Project Structure

```
k3s-certbot/
├── .github/
│   └── workflows/          # GitHub Actions workflows
│       └── docker-build.yml
├── docker/                 # Docker-related files
│   └── Dockerfile
├── k8s/
│   └── cert-manager/
│       ├── base/          # Kubernetes base configurations
│       │   ├── kustomization.yaml
│       │   ├── namespace.yaml
│       │   └── storage.yaml
│       └── templates/     # Configuration templates
│           ├── cronjob.yaml
│           └── secrets.yaml
├── scripts/
│   └── generate-k8s-config.sh
├── .gitignore
├── LICENSE
└── README.md
```

## Quick Start

1. Clone the repository:
```bash
git clone https://github.com/snis/k3s-certbot.git
cd k3s-certbot
```

2. Generate Kubernetes configurations:
```bash
./scripts/generate-k8s-config.sh
```
When prompted, enter:
- Your domain (e.g., example.com)
- Email address for Let's Encrypt notifications
- Loopia API credentials (e.g., userbot@loopiaapi, p4s$-the-w0rd)

3. Apply configurations to your K3s cluster:
```bash
kubectl apply -k k8s/cert-manager/base
```

4. Verify the installation:
```bash
kubectl get pods -n cert-manager
kubectl get cronjobs -n cert-manager
```

## Certificate Management

### Automatic Renewal
The CronJob is configured to run monthly to check and renew certificates as needed. No manual intervention is required.

### Manual Renewal
To trigger a manual certificate renewal:
```bash
kubectl create job --from=cronjob/certbot-renew manual-renew -n cert-manager
```

## Security Considerations

- Generated configuration files in `k8s/cert-manager/base/` contain sensitive data
- The `secrets.yaml` file contains API credentials - keep it secure
- Never commit secrets or generated configurations to git (they are gitignored by default in .gitignore)
- Store credentials securely and restrict access to configuration files

## Docker Image

The Certbot container image is automatically built and published to GitHub Container Registry (ghcr.io) when changes are pushed to the `docker/` directory.

To manually trigger a build, use the "Actions" tab in the GitHub repository and run the "Build Certbot Loopia Image" workflow.

## Troubleshooting

1. Check CronJob status:
```bash
kubectl get cronjobs -n cert-manager
```

2. View logs from the latest job:
```bash
kubectl get pods -n cert-manager
kubectl logs <pod-name> -n cert-manager
```

3. Common issues:
   - Ensure Loopia API credentials are correct
   - Verify domain ownership and API access
   - Check if the certificate storage volume is properly mounted

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

Apache License 2.0 - see [LICENSE](LICENSE) for details
