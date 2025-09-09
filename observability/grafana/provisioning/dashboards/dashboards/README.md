# Grafana Dashboards

This directory contains Grafana dashboard definitions that will be automatically provisioned when Grafana starts.

## Adding New Dashboards

1. Export your dashboard from Grafana as JSON
2. Save the JSON file in this directory  
3. Commit the file to git
4. Argo CD will automatically sync the new dashboard

## Dashboard Guidelines

- Use meaningful names and descriptions
- Include appropriate tags for categorization
- Set appropriate refresh intervals
- Use template variables for flexibility
- Follow consistent naming conventions

## Default Dashboards

The observability stack includes default dashboards for:
- Victoria Metrics monitoring
- Kubernetes cluster overview
- Application performance monitoring
- Alertmanager status

Custom dashboards should be added here to ensure they are version controlled and automatically deployed.