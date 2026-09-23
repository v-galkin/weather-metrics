# Weathervane

A FastAPI service that pulls live weather data from a public API, exposes it as Prometheus metrics, and visualizes it in Grafana — with a full CI/CD pipeline via GitHub Actions.

**Stack:** FastAPI · httpx · APScheduler · prometheus_client · Prometheus · Grafana · Docker · GitHub Actions

---

## Architecture

```
OpenWeatherMap API
       ↓ (fetched periodically via APScheduler)
FastAPI app (fetch logic + /metrics endpoint)
       ↓ (scraped every N seconds)
Prometheus (stores time series)
       ↓ (queried)
Grafana (dashboards)
```

---

## Tasks

### 1. Core FastAPI app
- [ ] Scaffold project structure (`main.py`, `config.py`, `weather_client.py`)
- [ ] Set up OpenWeatherMap API key via environment variable
- [ ] Write `weather_client.py` — async `httpx` call to fetch current weather for a saved location
- [ ] Add a plain JSON endpoint (e.g. `/weather/{location}`) to confirm the API integration works standalone
- [ ] Add `/health` endpoint returning 200

### 2. Prometheus metrics
- [ ] Add `prometheus_client` dependency
- [ ] Define Gauges in `metrics.py`:
  - [ ] `weather_temperature_celsius{location}`
  - [ ] `weather_humidity_percent{location}`
  - [ ] `weather_wind_speed_mps{location}`
- [ ] Add `/metrics` endpoint returning `generate_latest()` with correct content type
- [ ] Confirm real values appear when hitting `/metrics` manually

### 3. Scheduling
- [ ] Add `APScheduler` dependency
- [ ] Write `scheduler.py` — periodic async job that calls `weather_client` and updates Gauges
- [ ] Wire scheduler into FastAPI's startup/lifespan event
- [ ] Confirm metrics update automatically without manual triggering

### 4. Prometheus + Grafana
- [ ] Write `prometheus.yml` scrape config targeting the FastAPI service
- [ ] Confirm Prometheus is scraping successfully (check target status in Prometheus UI)
- [ ] Add Grafana, connect Prometheus as a data source
- [ ] Build dashboard panels:
  - [ ] Temperature over time (line graph)
  - [ ] Humidity over time
  - [ ] Wind speed over time
  - [ ] Multi-location comparison (if tracking more than one city)

### 5. Containerization
- [ ] Write `Dockerfile` for the FastAPI app
- [ ] Write `docker-compose.yml` with three services: `weather-service`, `prometheus`, `grafana`
- [ ] Confirm full stack runs end-to-end via `docker-compose up`

### 6. Testing
- [ ] Add `pytest` + `pytest-mock` (or `unittest.mock`)
- [ ] Unit test `weather_client.py` with a mocked API response (no real API calls in tests)
- [ ] Test `/metrics` returns valid Prometheus-format output
- [ ] Test `/health` returns 200

### 7. Linting
- [ ] Add `ruff` (or `flake8`) + `black`
- [ ] Confirm lint passes locally before wiring into CI

### 8. CI/CD (GitHub Actions) — Phase 1
- [ ] Create `.github/workflows/ci-cd.yml`
- [ ] Stage: run tests on push/PR
- [ ] Stage: run lint/format check
- [ ] Stage: build Docker image (only if tests + lint pass)
- [ ] Stage: push image to `ghcr.io` using built-in `GITHUB_TOKEN`
- [ ] Confirm pipeline runs green end-to-end on a push to `main`

### 9. CI/CD — Phase 2 (deploy to Oracle Cloud VPS)
- [ ] Provision the VPS: install Docker + Docker Compose
- [ ] Open the required ports — both the OS firewall (`iptables`/`firewalld`) **and** Oracle Cloud's own Security List/Network Security Group; Oracle Cloud blocks inbound traffic at the cloud level by default regardless of the OS firewall
- [ ] Copy `docker-compose.yml` (+ `prometheus.yml`, and Grafana provisioning if added) to the VPS
- [ ] Store SSH access as GitHub Secrets (e.g. `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`) and `OPENWEATHER_API_KEY` as a secret too — never committed
- [ ] Add deploy stage: SSH into the VPS, `docker compose pull` the new image, `docker compose up -d`
- [ ] Confirm automatic deploy on push to `main`
- [ ] Decide what's actually public: the FastAPI app (and maybe Grafana) yes; Prometheus and raw `/metrics` probably kept internal-only or behind auth
- [ ] Consider a reverse proxy (nginx/Caddy) with TLS if this stays up long-term

### 10. Documentation
- [ ] Write README with architecture diagram, setup instructions, and design decisions
- [ ] Record a short asciinema/terminal demo (service running + metrics updating)
- [ ] Screenshot/record the Grafana dashboard under live data
- [ ] Add `NOTES.md` capturing problems hit + how they were solved

---

## Suggested build order
1. Core FastAPI app (standalone JSON endpoint)
2. Prometheus metrics + `/metrics` endpoint
3. Scheduling (APScheduler)
4. Prometheus + Grafana wiring
5. Containerization (`docker-compose`)
6. Tests + linting
7. CI/CD Phase 1
8. Documentation
9. CI/CD Phase 2 (deploy to Oracle Cloud VPS)
