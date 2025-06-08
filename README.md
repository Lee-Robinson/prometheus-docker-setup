# Prometheus Docker Setup

A simple, production-ready Prometheus monitoring setup using Docker Compose with embedded configuration. This eliminates common file mounting issues and makes deployment through Portainer seamless.

## Features

- ✅ Self-contained configuration (no external config files needed)
- ✅ Persistent data storage with Docker volumes
- ✅ Ready for Portainer deployment
- ✅ Easy to extend with additional monitoring services
- ✅ Production-ready with proper retention settings

## Quick Start

### Method 1: Deploy via Portainer (Recommended)

1. Log into your Portainer instance
2. Navigate to **Stacks** → **Add Stack**
3. Name your stack (e.g., `prometheus-monitoring`)
4. Copy the contents of `docker-compose.yml` into the editor
5. Click **Deploy the stack**
6. Access Prometheus at `http://your-server-ip:9090`

### Method 2: Deploy via Command Line

```bash
# Clone this repository
git clone https://github.com/Lee-Robinson/prometheus-docker-setup.git
cd prometheus-docker-setup

# Deploy the stack
docker compose up -d

# Check status
docker compose ps
```

## Configuration

The Prometheus configuration is embedded directly in the compose file under the `configs` section. This approach:

- Eliminates file mounting issues
- Makes the setup truly portable
- Reduces deployment complexity
- Works seamlessly in container orchestration environments

### Default Configuration

The setup includes:
- **Global scrape interval**: 15 seconds
- **Data retention**: 200 hours
- **Self-monitoring**: Prometheus monitors itself
- **Web interface**: Enabled with lifecycle management

### Modifying Configuration

To modify the Prometheus configuration:

1. Edit the `content` section under `prometheus_config` in `docker-compose.yml`
2. Redeploy the stack (Portainer) or run `docker compose up -d`

## Accessing Prometheus

Once deployed, access the Prometheus web interface at:
```
http://your-server-ip:9090
```

### Basic Queries to Try

- `up` - Shows target status
- `prometheus_build_info` - Version information
- `rate(prometheus_http_requests_total[5m])` - HTTP request rate

## Data Persistence

Prometheus data is stored in a Docker volume named `prometheus-data`, ensuring your metrics persist across container restarts and updates.

## Extending the Setup

### Adding Node Exporter (System Metrics)

To monitor system metrics, add this service to your compose file:

```yaml
  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    ports:
      - "9100:9100"
    command:
      - '--path.rootfs=/host'
    volumes:
      - '/:/host:ro,rslave'
    restart: unless-stopped
```

Then add this scrape config:
```yaml
- job_name: 'node_exporter'
  static_configs:
    - targets: ['node_exporter:9100']
```

### Adding cAdvisor (Container Metrics)

For Docker container monitoring:

```yaml
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    restart: unless-stopped
```

## Troubleshooting

### Common Issues

**Container keeps restarting**
- Check logs: `docker compose logs prometheus`
- Verify configuration syntax in the compose file

**Can't access web interface**
- Ensure port 9090 is not blocked by firewall
- Check container status: `docker compose ps`

**Configuration changes not applying**
- Redeploy the entire stack to pick up config changes
- Use `docker compose down && docker compose up -d`

## Security Considerations

For production deployments:

1. **Firewall**: Restrict access to port 9090
2. **Authentication**: Consider adding reverse proxy with auth
3. **TLS**: Enable HTTPS for web interface
4. **Network**: Use Docker networks to isolate services

## Contributing

Feel free to submit issues and enhancement requests!

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Portainer Documentation](https://docs.portainer.io/)
