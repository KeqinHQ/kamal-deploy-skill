# Docker mirrors for China

There are **two distinct kinds of mirrors** in this world. Don't confuse them:

1. **Install mirror** — an apt/yum repo that hosts the `docker-ce` packages themselves. Used once during install.
2. **Registry mirror** — a Docker Hub mirror configured in `/etc/docker/daemon.json`. Used at runtime for every `docker pull` from `docker.io`.

You typically need both.

Last reviewed: 2026-04. If a mirror in this list stops working, try another and update this file.

---

## Install mirrors (apt/yum repos)

### Aliyun
Official docs: https://developer.aliyun.com/mirror/docker-ce

**Ubuntu / Debian:**
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**CentOS / RHEL:**
```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Aliyun has different network paths for ECS:
- Public internet: `https://mirrors.aliyun.com/`
- ECS VPC (faster from inside Aliyun): `http://mirrors.cloud.aliyuncs.com/`
- ECS Classic: `http://mirrors.aliyuncs.com/`

### Tencent Cloud
Official docs: https://cloud.tencent.com/document/product/213/46000

**Ubuntu / Debian:**
```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**CentOS:**
```bash
sudo yum-config-manager --add-repo=https://mirrors.cloud.tencent.com/docker-ce/linux/centos/docker-ce.repo
sudo sed -i "s/download.docker.com/mirrors.tencentyun.com\/docker-ce/g" /etc/yum.repos.d/docker-ce.repo
sudo yum install -y docker-ce
```

### Quick path: official installer with built-in mirror flag
Works on any Linux but only if `get.docker.com` itself is reachable:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

---

## Registry mirrors (for /etc/docker/daemon.json)

These go in `daemon.json` and affect every Docker Hub pull at runtime.

### Aliyun container registry mirror — most stable
- URL pattern: `https://<your-id>.mirror.aliyuncs.com`
- Get yours at: https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
- Requires a free Aliyun account; per-user URL — each account gets a personal mirror endpoint
- Recommended for production

### Tencent Cloud
- URL: `https://mirror.ccs.tencentyun.com`
- Only accessible from inside Tencent Cloud's network (CVM, etc.) — won't work from other clouds
- No authentication needed if you're on a Tencent VPC

### Public mirrors (may rotate, no account needed)
- `https://docker.m.daocloud.io` — DaoCloud public; usually works but availability fluctuates
- `https://docker.1ms.run`
- `https://hub.rat.dev`
- `https://docker.anyhub.us.kg`

These come and go. For a production setup, don't rely on a single public mirror — use Aliyun, your cloud's own mirror, or self-host a registry pull-through cache.

### daemon.json example

```json
{
  "registry-mirrors": [
    "https://<your-aliyun-id>.mirror.aliyuncs.com",
    "https://docker.m.daocloud.io"
  ],
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" }
}
```

List multiple mirrors for fallback — Docker tries them in order. After editing:

```bash
sudo systemctl restart docker
docker info | grep -A 3 "Registry Mirrors"
```

---

## Notes on private registries

The team's standard private registry for Kamal app images is `https://private-registry.tealight.uk:8443/` — this is where your built image gets pushed during `kamal deploy`. Configure it in `deploy.yml`:

```yaml
image: <your-app-image-name>
registry:
  server: private-registry.tealight.uk:8443
  username:
    - KAMAL_REGISTRY_USERNAME
  password:
    - KAMAL_REGISTRY_PASSWORD
```

Credentials live in `.kamal/secrets`. Ask a teammate if you don't have them.

You don't need a registry mirror for the team's private registry — the mirror in `daemon.json` only affects pulls from `docker.io` (like `basecamp/kamal-proxy`). Pulls from `private-registry.tealight.uk:8443` go directly to that registry, which is the correct behavior.

### Cloud-specific alternatives

If you can't use the team registry for some reason, Chinese clouds offer their own:
- Aliyun Container Registry (ACR): `registry.cn-hangzhou.aliyuncs.com` (or other region)
- Tencent Container Registry (TCR): `<id>.tencentcloudcr.com`

Same `deploy.yml` shape, just swap the `server` value and grab credentials from that cloud's console.
