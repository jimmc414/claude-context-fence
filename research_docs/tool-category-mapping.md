# Tool Category Mapping

Systematic assignment of all ~245 tools from the inventory to task-based categories.

## Category Assignments

### task-json (Structured Data: JSON, YAML, XML)
```
jq, yq, xq, jo, jd, gron, dasel, fx, jless, xmlstarlet, jsonlint, yamllint, dyff
```
**Count: 13**

### task-tabular (CSV, TSV, Columns, Aggregation)
```
awk, cut, sort, uniq, column, paste, join, comm, tsort,
miller (mlr), datamash, qsv, xsv, q, textql,
numaverage, numbound, numgrep, numinterval, numnormalize,
numprocess, numrandom, numrange, numround, numsum
```
**Count: 25**

### task-text (Search, Filter, Transform Text)
```
grep, rg (ripgrep), ag, ast-grep, sed, tr, awk, sd,
head, tail, tac, rev, nl, wc, fmt, fold, expand, unexpand,
split, csplit, pr, iconv, recode, strings
```
**Count: 24**

### task-files (Find, Compare, Sync, Disk)
```
find, fd (fdfind), locate, updatedb, bfs, which, whereis,
file, stat, readlink, realpath, namei,
ls, tree,
diff, diff3, sdiff, cmp, difft, difftastic, wiggle, patch,
cp, mv, rm, ln, mkdir, rmdir, touch, install, shred, rename,
rsync, scp, rclone, sshfs,
du, df, ncdu, dust, duf,
lsblk, findmnt, mount, umount, blkid, losetup, fallocate, wipefs
```
**Count: 50**

### task-http (HTTP Requests, Downloads, APIs)
```
curl, wget, xh, httpie, aria2c,
hurl, prism, json-server
```
**Count: 8**

### task-media (Video, Audio, Images, Documents)
```
ffmpeg, ffprobe, yt-dlp,
imagemagick (convert, identify, mogrify, composite),
exiftool, gifsicle, optipng, jpegoptim,
sox, whisper,
pandoc, weasyprint, unoconv,
graphviz (dot), mermaid-cli
```
**Count: 16**

### task-archives (Compression, Archiving)
```
tar, gzip, gunzip, bzip2, bunzip2, xz, unxz, zstd,
zip, unzip, 7z, rar, unrar,
pigz, cpio, ar, zrun
```
**Count: 17**

### task-process (Process Management, Job Control)
```
ps, top, htop, btop, ctop,
kill, pkill, killall, skill, snice,
pgrep, pidof, pidwait,
jobs, bg, fg, nohup, disown,
timeout, watch, xargs, parallel,
nice, renice, ionice, chrt, taskset, prlimit,
lsof, fuser, pmap, pwdx, pstree
```
**Count: 34**

### task-system (System Info, Services, Logs, Scheduling)
```
uname, hostname, uptime, whoami, id, groups, users, who, w,
free, vmstat, iostat, mpstat, sar, tload,
lscpu, lsmem, lsns, lshw, lspci, lsusb, sensors, nvidia-smi,
dmesg, journalctl, logger, ccze, logrotate, angle-grinder,
systemctl, service, sysctl,
cron, crontab, at,
date, cal, timedatectl, hwclock,
dateadd, datediff, dateround, dateseq, datesort, datetest, datezone, strptime
```
**Count: 48**

### task-network (Connectivity, DNS, Ports, Diagnostics)
```
ping, traceroute, tracepath, mtr,
dig, nslookup, host, whois, dnsutils,
ss, netstat, ip, ifconfig, route, arp, arpd,
nmap, nc (netcat), ngrep, tcpdump, tshark,
ip, tc, bridge, nstat, devlink, lnstat, rtmon, ctstat, routel, genl, rdma,
iptables, nft,
ssh, autossh, mosh, sshuttle, bore
```
**Count: 40**

### task-permissions (Users, Groups, Permissions)
```
chmod, chown, chgrp, chroot,
id, whoami, groups,
su, sudo, passwd,
useradd, usermod, userdel, groupadd, groupmod,
getfacl, setfacl, umask
```
**Count: 18**

### task-git (Version Control)
```
git, gh, glab,
delta, difftastic, tig, lazygit, gitui,
git-lfs, git-crypt, git-extras (60+ subcommands),
pre-commit, gitleaks
```
**Count: 13 (but git-extras adds 60+)**

### task-dev (Build, Lint, Format, Languages)
```
make, cmake, ninja, meson, just, invoke,
gcc, g++, clang, ldd,
shellcheck, shfmt,
eslint, prettier, ruff, black, pyright, mypy, bandit, semgrep,
hadolint, actionlint,
python, pip, pipx, uv, poetry, pdm, pyenv, virtualenv,
node, npm, pnpm, yarn, npx, tsx, tsc, bun, deno, nvm, volta,
esbuild, vite,
cargo, go, java, javac
```
**Count: 48**

### task-crypto (Hashing, Encoding, Encryption)
```
md5sum, sha1sum, sha256sum, sha512sum, b2sum, cksum, rhash,
base64, base32, basenc, xxd, od, hexdump, uuencode,
openssl, gpg, gnupg, age, sops, pass, vault,
ssh-keygen, ssh-agent, mkcert, certbot
```
**Count: 25**

### task-database (SQL, Key-Value, Data Stores)
```
sqlite3, sqlite-utils,
psql, mysql, mariadb,
redis-cli,
duckdb, datasette, meilisearch, dolt
```
**Count: 10**

### task-containers (Docker, Environments, Isolation)
```
docker, docker-compose, dive, ctop,
direnv, mise,
screen, tmux, zellij,
unshare, nsenter, chroot, setsid,
lima, devcontainer
```
**Count: 15**

### task-debug (Tracing, Profiling, Memory)
```
strace, ltrace, dtrace, bpftrace,
perf, sysprof, flamegraph,
py-spy, scalene, memray, austin,
clinic, 0x,
valgrind, massif, heaptrack,
gdb, lldb,
time, hyperfine
```
**Count: 20**

### task-watch (File Watching, Triggers)
```
inotifywait, fswatch, watchman, entr,
watch (also in task-process)
```
**Count: 5**

### task-api-test (Load Testing, API Validation)
```
k6, artillery, vegeta, hey, ab, wrk, wrk2,
hurl, schemathesis,
prism
```
**Count: 10**

---

## Tools Intentionally Excluded from Skills

### Cloud CLIs (require account setup, have extensive own docs)
```
aws, az, gcloud, wrangler
```

### AI/LLM Tools (rapidly evolving, model-specific)
```
ollama, llm, openai, anthropic, huggingface-hub, transformers-cli,
mlflow, dvc, bentoml, torchserve, triton, vllm, text-generation-inference,
gpt4all, llamacpp, coqui-tts
```

### Package Managers (well-documented, distro-specific)
```
apt, dpkg, dnf, yum, pacman, brew, nix
```

### Code Generation (project-specific)
```
quicktype, openapi-generator, protoc, buf, graphql-codegen, cookiecutter
```

### Testing Frameworks (language-specific, well-documented)
```
pytest, jest, vitest, hypothesis, playwright
```

### Trivial/Self-Explanatory
```
echo, printf, yes, seq, true, false, sleep, env, printenv, pwd,
basename, dirname, cat, less, more, tee, cd, read, test, factor, shuf
```

### Interactive/TUI Only (not agent-friendly)
```
vim, ed, fzf, pet, glow, cheat
```

### Clipboard (platform-specific)
```
xclip, pbcopy, pbpaste, wl-copy, wl-paste
```

### Misc Specialized
```
expect (interactive automation), ntfy (notifications),
j2cli (jinja), mo (mustache), envsubst (env substitution)
```

---

## Summary

| Category | Tool Count | Priority |
|----------|-----------|----------|
| task-json | 13 | High |
| task-tabular | 25 | High |
| task-text | 24 | High |
| task-files | 50 | High |
| task-http | 8 | High |
| task-media | 16 | Medium |
| task-archives | 17 | Medium |
| task-process | 34 | Medium |
| task-system | 48 | Medium |
| task-network | 40 | Medium |
| task-permissions | 18 | Low |
| task-git | 13+ | Medium |
| task-dev | 48 | Low |
| task-crypto | 25 | Low |
| task-database | 10 | Medium |
| task-containers | 15 | Medium |
| task-debug | 20 | Medium |
| task-watch | 5 | Low |
| task-api-test | 10 | Low |
| **TOTAL CATEGORIZED** | **~440** | |
| Excluded | ~60 | |

Note: Some tools appear in multiple categories (e.g., awk in task-text and task-tabular).
The count exceeds 245 because of this overlap and because some "tools" are actually suites.
