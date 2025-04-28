# aind-slurm-rest-v2

[![License](https://img.shields.io/badge/license-MIT-brightgreen)](LICENSE)
[![semantic-release: angular](https://img.shields.io/badge/semantic--release-angular-e10079?logo=semantic-release)](https://github.com/semantic-release/semantic-release)
![Python](https://img.shields.io/badge/python->=3.7-blue?logo=python)

### Usage

For use with Slurm v0.0.40 endpoints.

```python
from aind_slurm_rest_v2 import ApiClient as Client
from aind_slurm_rest_v2 import Configuration as Config
from aind_slurm_rest_v2.api.slurm_api import SlurmApi
from aind_slurm_rest_v2.api.slurmdb_api import SlurmdbApi
from aind_slurm_rest_v2.models.v0040_job_submit_req import V0040JobSubmitReq
from aind_slurm_rest_v2.models.v0040_job_desc_msg import V0040JobDescMsg
from aind_slurm_rest_v2.models.v0040_uint64_no_val import V0040Uint64NoVal
from aind_slurm_rest_v2.models.v0040_uint32_no_val import V0040Uint32NoVal
import os

host = "http://slurm2/api"
username = os.getenv("SLURM_USER")
access_token = os.getenv("SLURM_TOKEN")
config = Config(host=host, username=username, access_token=access_token)
slurm = SlurmApi(Client(config))

# ping test
ping_response = slurm.slurm_v0040_get_ping()
print(ping_response)

command_str = [
            "#!/bin/bash",
            "\necho",
            "'Hello World'",
            "&&",
            "sleep",
            "15",
            "&&",
            "echo",
            "'Goodbye'"
        ]
script = " ".join(command_str)

hpc_env = ["PATH=/bin:/usr/bin/:/usr/local/bin/", "LD_LIBRARY_PATH=/lib/:/lib64/:/usr/local/lib"]

job_props = V0040JobDescMsg(
  name = "test_job",
  partition = "aind",
  environment = hpc_env,
  standard_output = "/allen/aind/scratch/svc_aind_airflow/dev/logs/%x_%j.out",
  standard_error = "/allen/aind/scratch/svc_aind_airflow/dev/logs/%x_%j_error.out",
  current_working_directory=".",
  time_limit = V0040Uint32NoVal(set=True, number=1),
  memory_per_cpu = V0040Uint64NoVal(set=True, number=50),
  tasks = 1,
  minimum_cpus = 1,
  maximum_nodes = 1,
)

job_submission = V0040JobSubmitReq(script=script, job=job_props)
submit_response = slurm.slurm_v0040_post_job_submit(v0040_job_submit_req=job_submission)
job_id = str(submit_response.job_id)
job_response = slurm.slurm_v0040_get_job(job_id=job_id)
print(job_response)
# If the job has been cleared from the short-term cache, info can be pulled
# from the db
slurmdb = SlurmdbApi(Client(config))
db_job_response = slurmdb.slurmdb_v0040_get_job(job_id=job_id)

```

## Installation
The code is automatically generated using openapi tools and the specification from slurm.

### Please reach out to Central IT for the `openapi.json` file.

### To create the python code, openapi tools is used. `generateSourceCodeOnly` in `configs.json` can be set to `False` to generate tests and additional files.
```bash
docker run --rm \
  -u "$(id -u):$(id -g)" \
  -v ${PWD}:/local openapitools/openapi-generator-cli:latest generate \
  --skip-validate-spec \
  --config /local/configs.json \
  -i /local/openapi.json \
  -g python \
  -o /local/src
```

## Contributing

### Pull requests

For internal members, please create a branch. For external members, please fork the repository and open a pull request from the fork. We'll primarily use [Angular](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit) style for commit messages. Roughly, they should follow the pattern:
```text
<type>(<scope>): <short summary>
```

where scope (optional) describes the packages affected by the code changes and type (mandatory) is one of:

- **build**: Changes that affect build tools or external dependencies (example scopes: pyproject.toml, setup.py)
- **ci**: Changes to our CI configuration files and scripts (examples: .github/workflows/ci.yml)
- **docs**: Documentation only changes
- **feat**: A new feature
- **fix**: A bugfix
- **perf**: A code change that improves performance
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **test**: Adding missing tests or correcting existing tests

### Semantic Release

The table below, from [semantic release](https://github.com/semantic-release/semantic-release), shows which commit message gets you which release type when `semantic-release` runs (using the default configuration):

| Commit message                                                                                                                                                                                   | Release type                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| `fix(pencil): stop graphite breaking when too much pressure applied`                                                                                                                             | ~~Patch~~ Fix Release, Default release                                                                          |
| `feat(pencil): add 'graphiteWidth' option`                                                                                                                                                       | ~~Minor~~ Feature Release                                                                                       |
| `perf(pencil): remove graphiteWidth option`<br><br>`BREAKING CHANGE: The graphiteWidth option has been removed.`<br>`The default graphite width of 10mm is always used for performance reasons.` | ~~Major~~ Breaking Release <br /> (Note that the `BREAKING CHANGE: ` token must be in the footer of the commit) |
