# DriftGuard - Terraform Drift Detection Dashboard

![DriftGuard Dashboard](https://customer-assets.emergentagent.com/job_0c792fcf-4fcd-45be-9a1f-6923490aae21/artifacts/k5lf1y7p_image.png)

A comprehensive infrastructure monitoring dashboard for detecting and visualizing Terraform state drift across workspaces. Built for DevOps teams managing infrastructure at scale.

## Features

### Dashboard Overview
- **Real-time Stats**: Prod Drift, Total Drift, Tier-0 Drift, Success Rate
- **Plan Summary**: Aggregated additions, changes, and destroys across all workspaces
- **Drift Trend Chart**: 30-day visualization by environment (Prod/Staging/Dev)
- **Drift Distribution**: Bar chart showing drift by environment

### Workspaces Management
- Full workspace table with sorting capabilities
- Advanced filtering (Team, Environment, Criticality)
- Quick tabs: All | Drift | Synced
- Drill-down to individual workspace details
- Direct links to Terraform Cloud runs

### Alert System
- Severity-based alerts (Critical, Warning, Info)
- Alert acknowledgment workflow
- Configurable alert rules with toggle on/off
- Channel routing (Slack, Teams, ServiceNow, Email)

### Settings & Monitoring
- Runner health status
- System configuration display
- Detection settings overview
- Prometheus metrics reference

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 19, Tailwind CSS, Shadcn/UI, Recharts |
| Backend | FastAPI (Python) |
| Database | MongoDB |
| Charts | Recharts |

## Project Structure

```
drift_monitor/
├── backend/
│   ├── server.py          # FastAPI application with all endpoints
│   ├── requirements.txt   # Python dependencies
│   └── .env              # Environment variables
├── frontend/
│   ├── src/
│   │   ├── components/   # Reusable UI components
│   │   │   ├── Sidebar.jsx
│   │   │   ├── StatCard.jsx
│   │   │   ├── WorkspaceTable.jsx
│   │   │   ├── DriftChart.jsx
│   │   │   ├── AlertsList.jsx
│   │   │   └── EnvMetricsChart.jsx
│   │   ├── pages/        # Page components
│   │   │   ├── Dashboard.jsx
│   │   │   ├── Workspaces.jsx
│   │   │   ├── WorkspaceDetail.jsx
│   │   │   ├── Alerts.jsx
│   │   │   └── Settings.jsx
│   │   ├── App.js
│   │   ├── App.css
│   │   └── index.css
│   ├── package.json
│   └── .env
└── README.md
```

## Getting Started

### Prerequisites
- Node.js 18+
- Python 3.9+
- MongoDB

### Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set environment variables
cp .env.example .env
# Edit .env with your MongoDB connection string

# Run the server
uvicorn server:app --host 0.0.0.0 --port 8001 --reload
```

### Frontend Setup

```bash
cd frontend

# Install dependencies
yarn install

# Set environment variables
cp .env.example .env
# Edit .env with your backend URL

# Run the development server
yarn start
```

### Environment Variables

**Backend (.env)**
```env
MONGO_URL=mongodb://localhost:27017
DB_NAME=drift_monitor
CORS_ORIGINS=*
```

**Frontend (.env)**
```env
REACT_APP_BACKEND_URL=http://localhost:8001
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/dashboard/stats` | Aggregated dashboard statistics |
| GET | `/api/workspaces` | List workspaces with filters |
| GET | `/api/workspaces/:id` | Single workspace details |
| GET | `/api/metrics/trends` | Drift trend data (30 days) |
| GET | `/api/metrics/by-team` | Metrics grouped by team |
| GET | `/api/metrics/by-environment` | Metrics by environment |
| GET | `/api/alerts` | List alerts with filters |
| POST | `/api/alerts/:id/acknowledge` | Acknowledge an alert |
| GET | `/api/alert-rules` | List alert rules |
| POST | `/api/alert-rules/:id/toggle` | Toggle alert rule |
| GET | `/api/runner/health` | Runner health status |
| GET | `/api/filters/teams` | Available teams |
| GET | `/api/filters/environments` | Available environments |
| GET | `/api/filters/criticalities` | Available criticality levels |

## Prometheus Metrics

When integrated with Pushgateway, the following metrics are published:

```promql
# Core Metrics
terraform_drift_detected{org, workspace, env, team, app, criticality} = 0|1
terraform_drift_run_success{...} = 0|1
terraform_drift_run_duration_seconds{...} = X

# Plan Summary Metrics
terraform_plan_add{...} = N
terraform_plan_change{...} = N
terraform_plan_destroy{...} = N
```

## Kubernetes Deployment

### CronJob Configuration (Production)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: terraform-drift-detector
spec:
  schedule: "0 */1 * * *"  # Every hour for prod
  concurrencyPolicy: Forbid
  startingDeadlineSeconds: 600
  successfulJobsHistoryLimit: 1
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      activeDeadlineSeconds: 3600
      template:
        spec:
          containers:
          - name: drift-detector
            image: your-registry/drift-detector:latest
            env:
            - name: TFC_TOKEN
              valueFrom:
                secretKeyRef:
                  name: terraform-cloud
                  key: token
            - name: PUSHGATEWAY_URL
              value: "pushgateway.monitoring.svc.cluster.local:9091"
          restartPolicy: OnFailure
```

## Alert Routing

| Environment | Severity | Channel |
|-------------|----------|---------|
| Production | Critical | ServiceNow + PagerDuty |
| Production | Warning | Slack |
| Non-Prod | All | Microsoft Teams |
| Tier-0 | Any | On-call paging |

## Current Mode

> **Note**: This dashboard is currently running in **Mock Mode** with simulated data. To enable real drift detection:
> 1. Configure `TFC_TOKEN` environment variable
> 2. Set up Prometheus Pushgateway
> 3. Deploy the Kubernetes CronJob

## Roadmap

- [ ] Real Terraform Cloud API integration
- [ ] Prometheus/Pushgateway metrics collection
- [ ] Slack/Teams webhook notifications
- [ ] ServiceNow incident creation
- [ ] Historical drift analysis
- [ ] Resource-type breakdown
- [ ] Custom alert rule builder
- [ ] Export reports (CSV, PDF)

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

MIT License - see [LICENSE](LICENSE) for details.

---

**DEVOPS Lab** | Built for infrastructure teams managing Terraform at scale
