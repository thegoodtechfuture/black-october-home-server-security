# Black October Home Server Patch Check

Simple copy-paste commands to check whether your home server services are patched against the October 2026 ("Black October") vulnerabilities, and how to fix them if not.

*Last updated: October 29, 2026*

**Before you start:**
- Find your container names with `docker ps`. Replace `<container>` below with yours.
- You may need `sudo` in front of `docker` commands.
- In the update commands, `<service>` is the name under `services:` in your `docker-compose.yml`. Run them in the folder that holds that file.
- Updating your host (`apt upgrade`) does **not** update your containers. Each container has to be updated on its own.

---

## 1. Plex (actively exploited, update now)

**Check:**
```
curl -s http://localhost:32400/identity
```
Look at `version="..."` in the output.

**Patched:** 1.43.3.10861 or newer

**Fix:**
- Docker: `docker compose pull <service> && docker compose up -d <service>`
- NAS or native: update from your NAS app center, package manager, or Plex Web → Settings

**Already hacked?** Updating doesn't remove malware. Check:
```
ps aux | grep -Ei 'myawesomeproxy|mylovelyminer|x12345'
```
If anything shows up, wipe and rebuild the server and change your Plex password.

---

## 2. PHP apps: Nextcloud, WordPress, phpMyAdmin, etc. (under mass attack)

The bug is in PHP itself, so updating the app alone does not fix it.

**Check:**
```
docker exec <container> php -v
```
Nextcloud AIO users: the container is `nextcloud-aio-nextcloud`.

**Patched (match your version's first two numbers):**
| If you have | You need |
|---|---|
| 8.2.x | 8.2.35+ |
| 8.3.x | 8.3.36+ |
| 8.4.x | 8.4.27+ |
| 8.5.x | 8.5.12+ |
| 8.1 or older | No fix. Move to a newer image. |

**Fix:**
- Nextcloud AIO: go to `https://<server-ip>:8080` → Stop containers → Backup → Start and update containers. Don't use Nextcloud's own updater, it doesn't update PHP.
- Other Docker: `docker compose pull <service> && docker compose up -d <service>`

---

## 3. NGINX (patched, no known attacks yet)

Includes reverse proxies and apps that bundle nginx inside.

**Check:**
```
docker exec <container> nginx -v
```
Native install: `nginx -v`

**Patched:** 1.30.6 or newer, or 1.31.7 or newer
(1.31.0 through 1.31.6 are **not** patched)

**Fix:**
- Docker: `docker compose pull <service> && docker compose up -d <service>`
- Native: `sudo apt update && sudo apt upgrade nginx`

**Nginx Proxy Manager** shows an `openresty/...` version instead. Update to the latest NPM image and check its release notes.

---

## 4. Tailscale (patched, only matters if you use Tailscale SSH)

**Check:**
```
tailscale version
```

**Patched:** 1.104.4 or newer

**Fix:** update through your package manager or the Tailscale app. If you don't use Tailscale SSH, turn it off: `sudo tailscale set --ssh=false`

---

## 5. Node.js apps (patched, no known attacks yet)

Many apps run on Node, such as Uptime Kuma, n8n and Immich.

**Check:**
```
docker exec <container> node -v
```

**Patched:** 22.23.4+, 24.22.1+, or 26.12.1+ (match your major version)

**Fix:** `docker compose pull <service> && docker compose up -d <service>`
If the app hasn't released a new image yet, check back later.

---

## 6. GitLab self-hosted (critical, only if you run it)

**Check:**
```
docker exec <container> cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
```

**Patched:** 18.11.13, 19.0.10, 19.1.9, 19.2.9, 19.3.6, 19.4.3, 19.5.1 or newer (match your version)

**Fix:** `docker compose pull <service> && docker compose up -d <service>`

---

## 7. LiteLLM (malicious release, only if you run it)

**Check:**
```
docker exec <container> pip show litellm | grep Version
```

**Bad version:** exactly **1.107.0**. It was a credential stealer published on October 21.

**Fix:** update to a newer version, then **change every API key and password** that container could see. Updating alone doesn't undo stolen keys.

---

## Best single protection

Don't expose services like Plex or Nextcloud directly to the internet with port forwarding. Reach them through a VPN like Tailscale or WireGuard instead. Every attack being used right now needs direct access to the service.

---

## Sources

- [Plex2Shell (CISA KEV)](https://bleepingcomputer.com/news/security/cisa-hackers-chain-plex-flaws-in-plex2shell-attacks-to-breach-servers)
- [PHP2Shell patches](https://bleepingcomputer.com/news/security/php-ships-emergency-patches-for-critical-php2shell-flaws)
- [NGINX October 2026 fixes](https://braindetox.kr/en/posts/nginx_security_advisories_oct_2026.html)
- [Tailscale Tailworm fix (TS-2026-014)](https://hol.org/blog/tailscale-tailworm-ssh-root-escalation-ts-2026-014-upgrade-1-104-4)
- [Node.js October 2026 security release](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases)
- [GitLab 19.5.1 patch release](https://docs.gitlab.com/releases/patches/patch-release-gitlab-19-5-1-released/)
- [LiteLLM 1.107.0 incident](https://docs.litellm.ai/blog/security-incident-oct-21-2026)

*This is a community checklist, not official guidance. Always confirm with each project's own advisory.*
