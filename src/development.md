## Logging & tracing

The app is expected to be using `tracing` and `tracing-opentelemetry` to provide observability. Here's an example
boilerplate code to initialize tracing: https://github.com/DCNick3/batts/blob/main/backend/src/init_tracing.rs.

With it, when deployed into the environment providing the
relevant [OpenTelemetry environment variables](https://opentelemetry.io/docs/languages/sdk-configuration/general/), it
will automatically send traces to the configured collector.

For local development, usage of [jaeger](https://www.jaegertracing.io/) is recommended. It will collect traces and allow
to visualize them, without it going to the real observability platform.

To change the console logging level, set the `RUST_LOG` environment variable to the desired filter.
See [here](https://docs.rs/tracing-subscriber/latest/tracing_subscriber/filter/struct.EnvFilter.html) for more
information.

## Configuration

For configuring all the aspects of the application, including the passing of the sensitive keys, a simple framework
build on top of the [`config`](https://docs.rs/config/latest/config/) crate is used. The app expects to have a single
environment variable `ENVIRONMENT` to be set to either `dev` or `prod`. Then the app will layer the following config
files, in this order, to obtain the final configuration:

- `config.yaml` (common configuration, checked into the repository)
- `config.local.yaml` (common configuration, not checked into the repository, possible place for secrets)
- `config.${ENVIRONMENT}.yaml` (environment-specific configuration, checked into the repository)
- `config.${ENVIRONMENT}.local.yaml` (environment-specific configuration, not checked into the repository, possible
  place for secrets)
- from environment variables, with `.` being mangled as `__`, all uppercase, prefixed with `CONFIG_` (for
  example, `CONFIG_SERVER__ENDPOINT` for `server.endpoint`)

The following is the minimum configuration (might get expanded over time):

```yaml
apis:
  google_calendar:
    client_id: "your_client_id"
    client_secret: "your_client_secret"
  paypal:
    client_id: "your_client_id"
    client_secret: "your_client_secret"
  binance_pay:
    api_key: "your_api_key"
    secret: "your_secret"
db:
  url: "postgres://user:password@localhost/db"
```

To learn how to obtain the API keys and secrets, see the [Obtaining Tokens](./getting_tokens) page.

To learn how to use these APIs consult the [Using APIs](./apis/preface.md) section.

## Deployment

To deploy the app, it should be built into a Docker container. This can be done with a generic Rust `Dockerfile`:

```dockerfile
# syntax = docker/dockerfile:1.2

FROM bash AS get-tini

# Add Tini init-system
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /tini
RUN chmod +x /tini

FROM clux/muslrust:stable as build

ENV CARGO_INCREMENTAL=0

WORKDIR /volume
COPY . .

RUN --mount=type=cache,target=/root/.cargo/registry --mount=type=cache,target=/volume/target \
    cargo build --locked --profile ship --target x86_64-unknown-linux-musl && \
    cp target/x86_64-unknown-linux-musl/ship/awesome-leads-calendar /volume/awesome-leads-calendar

FROM gcr.io/distroless/static

LABEL org.opencontainers.image.source https://github.com/awesome-org/awesome-leads-calendar
EXPOSE 3000

ENV ENVIRONMENT=prod
ENV CONFIG_SERVER__ENDPOINT=0.0.0.0:3000

COPY --from=get-tini /tini /tini
COPY --from=build /volume/awesome-leads-calendar /awesome-leads-calendar
COPY config.yaml config.prod.yaml /

ENTRYPOINT ["/tini", "--", "/awesome-leads-calendar"]
```

This `Dockerfile` produces a very small container because Rust programs mostly don't depend on the system libraries and
hence the distroless image is used. The `tini` init system is used to handle signals properly.

The app will be deployed in a kubernetes cluster managed by an ArgoCD instance. The k8s manifest files go to
the `deployment` folder by convention. Then the following argocd application manifest can be used to deploy the app to
the cluster:

```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: awesome-leads-calendar
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/awesome-org/awesome-leads-calendar.git
    targetRevision: HEAD
    path: deployment
  destination:
    server: https://kubernetes.default.svc
    namespace: awesome-leads-calendar
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - Validate=false
```