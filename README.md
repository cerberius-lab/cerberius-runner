# Cerberius runner

Run the [Cerberius Agents](https://ai-agent.cerberius.com) support agent on your own systems.

Cerberius Agents investigates your support tickets with an AI agent, and drafts the reply for a
person to approve. Normally the agent runs in our cloud. With a **runner**, it runs in your own
Kubernetes cluster or on a server of yours, next to your systems:

- **Outgoing calls only.** The runner calls Cerberius Agents for work. You open no ports.
- **Your keys can stay with you.** Keep your AI key, repository keys, cluster login and database
  passwords on the runner. We never see them.
- **Changes need a person and your rules.** A change someone approves is signed by Cerberius
  Agents. Your runner checks the signature, then your own rules. Nothing is allowed until your
  rules allow it.
- **Offline means stop.** If the runner can't reach Cerberius Agents for 5 minutes, it stops its
  agents.

This repository holds these instructions and the job that publishes the packages. It holds no
source code.

## Packages

| What | Where |
|---|---|
| Helm chart | `oci://ghcr.io/cerberius-lab/helm/cerberius-runner` |
| Runner image | `ghcr.io/cerberius-lab/cerberius-runner` |
| Agent image (the runner starts one for each ticket) | `ghcr.io/cerberius-lab/cerberius-agent` |

All three are public: your cluster or server needs no login to download them. The images are
built for x86-64 (amd64).

## Install

Start in Cerberius Agents: **Settings › Agent › Where it runs › Add a runner**. It shows the exact
commands, with your runner's key in them. The key is shown once.

**Kubernetes**, with Helm 3.14 or later:

```sh
helm install cerberius-runner oci://ghcr.io/cerberius-lab/helm/cerberius-runner \
  --namespace cerberius-runner --create-namespace \
  --set runnerKey=rnr_... \
  --set clusterName=my-cluster \
  --set cerberius.signingPublicKey=...
```

**One Linux server**, with Docker and Docker Compose. In an empty folder:

```sh
mkdir -p cerberius-runner/secrets && cd cerberius-runner
chmod 700 secrets && touch rules.yaml .env && chmod 600 .env
docker run --rm ghcr.io/cerberius-lab/cerberius-runner:<version> compose > docker-compose.yml
printf '%s\n' 'RUNNER_KEY=rnr_...' 'CLUSTER_NAME=my-server' 'SIGNING_PUBLIC_KEY=...' > .env
docker compose up -d
```

Within a minute, the runner shows **Online** in Settings. Then choose **Your own runner** there.

The full guide, with keys kept on the runner and the rules for changes:
[Run the agent on your own systems](https://ai-agent.cerberius.com/help/platform-setup/own-runner).

## Update

An older runner keeps working. Settings shows **Update available** with the command for how you
installed it:

```sh
helm upgrade cerberius-runner oci://ghcr.io/cerberius-lab/helm/cerberius-runner \
  --namespace cerberius-runner --reset-then-reuse-values
```

On one server, print the new compose file with the new version, as above, and run
`docker compose up -d` again. Your settings in `.env`, `rules.yaml` and `secrets/` stay.

## Security

Found a security problem? Please tell us through the
[contact page](https://ai-agent.cerberius.com/contact) rather than in a public issue.

---

© Cerberius Pty Ltd. The packages are for use with Cerberius Agents.
