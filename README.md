# Grafana OnCall

> Unofficial fork of [Grafana OnCall OSS](https://github.com/grafana/oncall). Grafana Labs does not maintain or endorse this repository.
>
> The goal is to keep OnCall working with current Grafana releases and to carry security fixes for the images and Helm dependencies shipped here. New functionality may be added later.

This tree includes security updates for the Helm chart, Docker Compose files, and CI images.

## Security fixes

Bundled dependencies below were moved off versions affected by these advisories.

Helm chart:

- ingress-nginx 4.1.4 to 4.15.1 (controller v1.15.1): CVE-2025-1974, CVE-2025-1097, CVE-2025-1098, CVE-2025-24513, CVE-2025-24514, CVE-2026-3288
- cert-manager v1.8.0 to v1.21.2: GHSA-8rvj-mm4h-c258, CVE-2025-68121
- Prometheus chart 25.8.2 (v2.48.1) to 29.37.0 (v3.15.0): GHSA-8rm2-7qqf-34qm, GHSA-fw8g-cg8f-9j28
- Grafana image 11.1.4 to 13.2.4: CVE-2025-4123, CVE-2026-17033, CVE-2026-15577, CVE-2026-33374, CVE-2026-93294, CVE-2026-95502, CVE-2026-92539
- MariaDB image pinned to the 2026-10-15 MariaDB 12.0.2 index instead of `latest`
- An existing Grafana is attached with `helm/external-grafana.yaml` only at 13.2.4 or newer.
- Redis 6.2.7 to 8.2.6: CVE-2025-49844, CVE-2026-23479, CVE-2026-23631, CVE-2026-25243
- RabbitMQ 3.12.0 to 3.13.7: CVE-2023-46118
- PostgreSQL 14.5.0 to 14.18.0: CVE-2025-1094

Docker Compose and CI images:

- Redis 7.0.15 to 8.2.6: CVE-2025-49844, CVE-2026-23479
- RabbitMQ 3.12.0 to 3.13.7: CVE-2023-46118
- MySQL 8.0.32 to 8.0.44: CVE-2025-53040, CVE-2025-53042, CVE-2025-53044, CVE-2025-53045, CVE-2025-53053, CVE-2025-53054, CVE-2025-53062, CVE-2025-53069
- PostgreSQL 14.4 to 14.19: CVE-2025-1094
- Prometheus to v3.15.0: GHSA-8rm2-7qqf-34qm, GHSA-fw8g-cg8f-9j28

## Grafana OnCall

<img width="400px" src="docs/img/logo.png">

[![Latest Release](https://img.shields.io/github/v/release/grafana/oncall?display_name=tag&sort=semver)](https://github.com/grafana/oncall/releases)
[![License](https://img.shields.io/github/license/grafana/oncall)](https://github.com/grafana/oncall/blob/dev/LICENSE)
[![Docker Pulls](https://img.shields.io/docker/pulls/grafana/oncall)](https://hub.docker.com/r/grafana/oncall/tags)
[![Slack](https://img.shields.io/badge/join%20slack-%23grafana-%2Doncall-brightgreen.svg)](https://slack.grafana.com/)
[![Build Status](https://github.com/grafana/oncall/actions/workflows/on-commits-to-dev.yml/badge.svg)](https://github.com/grafana/oncall/actions/workflows/on-commits-to-dev.yml)

Developer-friendly incident response with brilliant Slack integration.

<!-- markdownlint-disable MD013 MD033 -->
<table>
  <tbody>
    <tr>
    <td width="75%"><img src="docs/img/screenshot.png"></td>
      <td><div align="center"><a href="https://grafana.com/docs/oncall/latest/mobile-app/">Android & iOS</a>:<br><img src="docs/img/screenshot_mobile.png"></div></td>
    </tr>
  </tbody>
</table>
<!-- markdownlint-enable MD013 MD033 -->

- Collect and analyze alerts from multiple monitoring systems
- On-call rotations based on schedules
- Automatic escalations
- Phone calls, SMS, Slack, Telegram notifications

## Getting Started

> [!IMPORTANT]  
> These instructions attach to Grafana 13.2.4 or newer, or install that build locally. You must enable
> `externalServiceAccounts` and `auth.managed_service_accounts_enabled` on that Grafana, and leave
> `accessControlOnCall` off. The docker files and the bundled Helm Grafana already do this.
> To use a Grafana you already run, set `GRAFANA_API_URL` and do not enable the `with_grafana` profile.
> Helm users: install with `helm/external-grafana.yaml` (`grafana.enabled=false`).
> 13.2.4 includes the 2026-10-13 fixes. 13.2.0 through 13.2.3 do not.

We prepared multiple environments:

- [production](https://grafana.com/docs/oncall/latest/open-source/#production-environment)
- [developer](./dev/README.md)
- hobby (described in the following steps)

1. Download [`docker-compose.yml`](docker-compose.yml):

   ```bash
   curl -fsSL https://raw.githubusercontent.com/grafana/oncall/dev/docker-compose.yml -o docker-compose.yml
   ```

2. Set variables:

   ```bash
   echo "DOMAIN=http://localhost:8080
   # Remove 'with_grafana' below if you want to use existing grafana
   # Add 'with_prometheus' below to optionally enable a local prometheus for oncall metrics
   # e.g. COMPOSE_PROFILES=with_grafana,with_prometheus
   COMPOSE_PROFILES=with_grafana
   # to setup an auth token for prometheus exporter metrics:
   # PROMETHEUS_EXPORTER_SECRET=my_random_prometheus_secret
   # also, make sure to enable the /metrics endpoint:
   # FEATURE_PROMETHEUS_EXPORTER_ENABLED=True
   SECRET_KEY=my_random_secret_must_be_more_than_32_characters_long" > .env
   ```

3. (Optional) If you want to enable/setup the prometheus metrics exporter
(besides the changes above), create a `prometheus.yml` file (replacing
`my_random_prometheus_secret` accordingly), next to your `docker-compose.yml`:

   ```bash
   echo "global:
     scrape_interval:     15s
     evaluation_interval: 15s

   scrape_configs:
     - job_name: prometheus
       metrics_path: /metrics/
       authorization:
         credentials: my_random_prometheus_secret
       static_configs:
         - targets: [\"host.docker.internal:8080\"]" > prometheus.yml
   ```

   NOTE: you will need to setup a Prometheus datasource using `http://prometheus:9090`
   as the URL in the Grafana UI.

4. Launch services:

   ```bash
   docker-compose pull && docker-compose up -d
   ```

5. Provision the plugin (If you run Grafana outside the included docker files install the plugin before these steps):

   If you are using the included docker compose file use `admin`/`admin` credentials and `localhost:3000` to
   perform this task.  If you have configured Grafana differently adjust your credentials and hostnames accordingly.

   ```bash
   # Note: onCallApiUrl 'engine' and grafanaUrl 'grafana' use the name from the docker compose file.  If you are 
   # running your grafana or oncall engine instance with another hostname adjust accordingly. 
   curl -X POST 'http://admin:admin@localhost:3000/api/plugins/grafana-oncall-app/settings' -H "Content-Type: application/json" -d '{"enabled":true, "jsonData":{"stackId":5, "orgId":100, "onCallApiUrl":"http://engine:8080", "grafanaUrl":"http://grafana:3000"}}'
   curl -X POST 'http://admin:admin@localhost:3000/api/plugins/grafana-oncall-app/resources/plugin/install'
   ```

6. Start using OnCall, log in to Grafana with credentials
   as defined above: `admin`/`admin`

7. Enjoy! Check our [OSS docs](https://grafana.com/docs/oncall/latest/open-source/) if you want to set up
   Slack, Telegram, Twilio or SMS/calls through Grafana Cloud.

## Troubleshooting

Here are some API calls that can be made to help if you are having difficulty connecting Grafana and OnCall.
(Modify parameters to match your credentials and environment)

   ```bash
   # Use this to get more information about the connection between Grafana and OnCall
   curl -X GET 'http://admin:admin@localhost:3000/api/plugins/grafana-oncall-app/resources/plugin/status'
   ```

   ```bash
   # If you added a user or changed permissions and don't see it show up in OnCall you can manually trigger sync.
   # Note: This is called automatically when the app is loaded (page load/refresh) but there is a 5 min timeout so 
   # that it does not generate unnecessary activity.
   curl -X POST 'http://admin:admin@localhost:3000/api/plugins/grafana-oncall-app/resources/plugin/sync'
   ```

## Update version

To update your Grafana OnCall hobby environment:

```shell
# Update Docker image
docker-compose pull engine

# Re-deploy
docker-compose up -d
```

After updating the engine, you'll also need to click the "Update" button on the [plugin version page](http://localhost:3000/plugins/grafana-oncall-app?page=version-history).
See [Grafana docs](https://grafana.com/docs/grafana/latest/administration/plugin-management/#update-a-plugin) for more
info on updating Grafana plugins.

## Join community

[<img width="200px" src="docs/img/slack.png">](https://slack.grafana.com/)
[<img width="200px" src="docs/img/GH_discussions.png">](https://community.grafana.com/)

Have a question, comment or feedback? Don't be afraid to [open an issue](https://github.com/grafana/oncall/issues/new/choose)!

## Stargazers over time

[![Stargazers over time](https://starchart.cc/grafana/oncall.svg)](https://starchart.cc/grafana/oncall)

## Further Reading

- _Automated migration from other on-call tools_ - [Migrator](https://github.com/grafana/oncall/tree/dev/tools/migrators)
- _Documentation_ - [Grafana OnCall](https://grafana.com/docs/oncall/latest/)
- _Overview Webinar_ - [YouTube](https://www.youtube.com/watch?v=7uSe1pulgs8)
- _How To Add Integration_ - [How to Add Integration](https://github.com/grafana/oncall/tree/dev/engine/config_integrations/README.md)
- _Blog Post_ - [Announcing Grafana OnCall, the easiest way to do on-call management](https://grafana.com/blog/2021/11/09/announcing-grafana-oncall/)
- _Presentation_ - [Deep dive into the Grafana, Prometheus, and Alertmanager stack for alerting and on-call management](https://grafana.com/go/observabilitycon/2021/alerting/?pg=blog)
