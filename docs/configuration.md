---
title: Configuration
sidebar_position: 3
---

# Configuration 

Configuration search paths:

- .gensbom.yaml
- .gensbom/gensbom.yaml
- ~/.gensbom.yaml
- \<k\>/gensbom/gensbom.yaml

For a custom configuration location use `--config` flag with any command.

Configuration format and default values.

```yaml
output_directory: ${XDG_CACHE_HOME}/gensbom
scribe:
  auth:
    login-url: https://scribesecurity-production.us.auth0.com
    grant-type: client_credentials
    enable: true
    audience: api.production.scribesecurity.com
  url: https://api.production.scribesecurity.com
  enable: false
context:
  context-type: local
attest:
  config: ""
  default: sigstore
  cocosign: {}
filter:
  filter-regex:
  - .*\.pyc
  - .*\.git/.*
  - .*\.git\.*
  filter-purl: []
bom:
  normalizers:
    packagejson:
      enable: true
  formats:
  - cyclonedx-json
  env: []
  force: false
  components:
  - metadata
  - layers
  - packages
  - syft
  - files
  - dep
  - commits
  attach-regex: []
  git:
    auth: ""
    tag: ""
    branch: ""
    commit: ""
find:
  format: cyclonedx-json
  all: false
verify:
  input-format: attest-cyclonedx-json
  force: false
sign:
  format: attest-cyclonedx-json
  input-format: cyclonedx-json
  force: false
dev:
  profile-cpu: false
  profile-mem: false
  backwards: false
  insecure: true
  failonerror: true
```
