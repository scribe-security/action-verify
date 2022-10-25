# Configuration 

Configuration search paths:
- .gensbom.yaml
- .gensbom/gensbom.yaml
- ~/.gensbom.yaml
- \<k\>/gensbom/gensbom.yaml

For a custom configuration location use `--config` flag with any command.

Configuration format and default values.
```yaml
output-directory: $XDG_CACHE_HOME/gensbom
scribe:
  auth:
    login-url: https://scribesecurity-production.us.auth0.com
    client-id: '******'
    client-secret: '******'
    grant-type: client_credentials
    enable: true
    audience: api.production.scribesecurity.com
  url: https://api.production.scribesecurity.com
  enable: false
context:
  context-type: local
attest:
  config: ""
  name: ""
  default: sigstore
filter:
  filter-regex:
  - .*\.pyc
  - .*\.git/.*
  filter-purl: []
bom:
  normalizers:
    packagejson:
      enable: true
  format:
  - cyclonedx-json
  env: []
  force: false
  components:
  - metadata
  - layers
  - packages
  - files
  - dep
  attach-regex: []
find:
  format: cyclonedx-json
  all: false
verify:
  input-format: attest-cyclonedx-json
sign:
  format: attest-cyclonedx-json
  input-format: cyclonedx-json
  force: false
log:
  structured: false
  level: ""
  file: ""
dev:
  profile-cpu: false
  profile-mem: false
  backwards: false
  insecure: true
```