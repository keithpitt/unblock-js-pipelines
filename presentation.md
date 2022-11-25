---
author: "Unblock Conf 2022"
date: "@keithpitt"
---
`.pipeline.yml`

```yaml
steps:
  - label: ":rspec: RSpec"
    depends_on: linting
    command: bundle exec rspec
    artifact_paths: "log/rspec-*.xml"
    key: "rspec"
    env:
      RAILS_ENV: test
    plugins:
      - docker-compose#v3.9.0:
          run: app
          env:
            - BUILDKITE_JOB_ID
            - CI
            - RAILS_ENV
            - BUILDKITE_ANALYTICS_TOKEN

  - group: "Lint all the things"
    key: linting
    steps:
      - label: ":lint-roller: Linting text and markdown"
        plugins:
          - docker-compose#v3.9.0:
              run: vale
              config: ./vale/docker-compose.yml

      - label: ":lint-roller: Linting for insensitive words"
        command: npm run lint -y
        plugins:
          - docker#v3.7.0:
              image: "node:lts-alpine3.14"
      - label: "Validate YAML"
        command:
          - npm run -y validate-agent-attributes-yaml
          - npm run -y validate-environment-variables-yaml
        plugins:
          - docker#v3.7.0:
              image: "node:lts-alpine3.14"
      - label: ":lint-roller: :markdown: Linting the Markdown"
        command: npm run -y mdlint
        plugins:
          - docker#v3.7.0:
              image: "node:lts-alpine3.14"

      - label: ":snake: Linting markdown files for snake case"
        command: npx -y @ls-lint/ls-lint
        plugins:
          - docker#v3.7.0:
              image: "node:lts-alpine3.14"

```

---

`.pipeline.js`

```javascript
module.exports = {
  steps: [
    {
      label: ":rspec: RSpec",
      depends_on: "linting",
      command: "bundle exec rspec",
      artifact_paths: "log/rspec-*.xml",
      key: "rspec",
      env: {
        RAILS_ENV: "test"
      },
      plugins: [
        {
          docker-compose#v3.9.0: {
            run: "app",
            env: [
              "BUILDKITE_JOB_ID",
              "CI",
              "RAILS_ENV",
              "BUILDKITE_ANALYTICS_TOKEN"
            ]
          }
        }
      ]
    },
    {
      group: "Lint all the things",
      key: "linting",
      steps: [
        {
          label: ":lint-roller: Linting text and markdown",
          plugins: [ { docker-compose#v3.9.0: { run: "vale", config: "./vale/docker-compose.yml" } } ]
        },
        {
          label: ":lint-roller: Linting for insensitive words",
          command: "npm run lint -y",
          plugins: [ { docker#v3.7.0: { image: "node:lts-alpine3.14" } } ]
        },
        {
          label: "Validate YAML",
          command: [
            "npm run -y validate-agent-attributes-yaml",
            "npm run -y validate-environment-variables-yaml"
          ],
          plugins: [ { docker#v3.7.0: { image: "node:lts-alpine3.14" } } ]
        },
        {
          label: ":lint-roller: :markdown: Linting the Markdown",
          command: "npm run -y mdlint",
          plugins: [ { docker#v3.7.0: { image: "node:lts-alpine3.14" } } ]
        }
      ]
    }
  ]
}
```

---

`.pipeline.js`

```javascript
const bk = require("bk");

const dockerComposePlugin = new bk.Plugin("docker-compose", "3.9.0");
const docker = new bk.Plugin("docker", "3.9.0")({ image: "node:lts-alpine3.14" });

module.exports = {
  steps: [
    {
      label: ":rspec: RSpec",
      depends_on: "linting",
      command: "bundle exec rspec",
      artifact_paths: "log/rspec-*.xml",
      key: "rspec",
      env: {
        RAILS_ENV: "test"
      },
      plugins: [
          dockerComposePlugin({
              run: "app",
              env: [ "BUILDKITE_JOB_ID", "CI", "RAILS_ENV", "BUILDKITE_ANALYTICS_TOKEN" ]
          })
      ]
    },
    {
      group: "Lint all the things",
      key: "linting",
      steps: [
        {
          label: ":lint-roller: Linting text and markdown",
          plugins: [ dockerComposePlugin({ run: "vale", config: "./vale/docker-compose.yml" }) ]
        },
        {
          label: ":lint-roller: Linting for insensitive words",
          command: "npm run lint -y",
          plugins: [ docker ]
        },
        {
          label: "Validate YAML",
          command: [
            "npm run -y validate-agent-attributes-yaml",
            "npm run -y validate-environment-variables-yaml"
          ],
          plugins: [ docker ]
        },
        {
          label: ":lint-roller: :markdown: Linting the Markdown",
          command: "npm run -y mdlint",
          plugins: [ docker ]
        }
      ]
    }
  ]
}
```

---

`.pipeline.js`

```javascript
const bk = require("bk");

const dockerComposePlugin = new bk.Plugin("docker-compose", "3.9.0");

function dockerCommand(label, commands) {
    return {
        label: label,
        command: commands,
        plugins: [ new bk.Plugin("docker", "3.9.0")({ image: "node:lts-alpine3.14" }) ]
    }
}

module.exports = {
  steps: [
    {
      label: ":rspec: RSpec",
      depends_on: "linting",
      command: "bundle exec rspec",
      artifact_paths: "log/rspec-*.xml",
      key: "rspec",
      env: {
        RAILS_ENV: "test"
      },
      plugins: [
          dockerComposePlugin({
              run: "app",
              env: [ "BUILDKITE_JOB_ID", "CI", "RAILS_ENV", "BUILDKITE_ANALYTICS_TOKEN" ]
          })
      ]
    },
    {
      group: "Lint all the things",
      key: "linting",
      steps: [
        {
          label: ":lint-roller: Linting text and markdown",
          plugins: [ dockerComposePlugin({ run: "vale", config: "./vale/docker-compose.yml" }) ]
        },
        dockerCommand(":lint-roller: Linting for insensitive words", "npm run lint -y"),
        dockerCommand("Validate YAML", [
            "npm run -y validate-agent-attributes-yaml",
            "npm run -y validate-environment-variables-yaml"
        ]),
        dockerCommand(":lint-roller: :markdown: Linting the Markdown","npm run -y mdlint")
      ]
    }
  ]
}
```

---

`.pipeline.js`

```javascript
const bk = require("bk");

const rspec = require("commands/rspec");
const dockerCommand = require("commands/dockerCommand");

const dockerComposePlugin = new bk.Plugin("docker-compose", "3.9.0");

module.exports = {
  steps: [
    rspec({ dockerCompose: true, enableTestAnalytics: true, dependsOn: "linting" }),
    {
      group: "Lint all the things",
      key: "linting",
      steps: [
        {
          label: ":lint-roller: Linting text and markdown",
          plugins: [ dockerComposePlugin({ run: "vale", config: "./vale/docker-compose.yml" }) ]
        },
        dockerCommand(":lint-roller: Linting for insensitive words", "npm run lint -y"),
        dockerCommand("Validate YAML", [
            "npm run -y validate-agent-attributes-yaml",
            "npm run -y validate-environment-variables-yaml"
        ]),
        dockerCommand(":lint-roller: :markdown: Linting the Markdown","npm run -y mdlint")
      ]
    }
  ]
}
```

---

`pipeline.js`

```javascript
class SetDefaultPriority {
  constructor(priority) {
    this.priority = priority;
  },
  apply(pipeline) {
    pipeline.steps.forEach((step) => {
      if (step instanceof CommandStep) {
        step.priority = this.priority;
      }
    });
  }
}

const pipeline = new Pipeline({ transformers: [ new SetDefaultPriority(5) ] });

pipeline.addSteps(
  new CommandStep({
    label: ":go: go fmt",
    key: "test-go-fmt",
    command: ".buildkite/steps/test-go-fmt.sh"
  }),
  new WaitStep({ label: "wait", continueOnFailure: false }),
  new CommandStep({
    label: ":go: go build",
    key: "build",
    command: ".buildkite/steps/go-build.sh"
  })
)

module.exports = pipeline;
```

---

`pipeline.sh`

```bash
# https://buildkite.com/docs/pipelines/defining-steps#dynamic-pipelines

echo "steps:"

for test_dir in test/*/; do
  echo "  - command: \"run_tests "${test_dir}"\""
done
```

`pipeline.js`

```javascript
module.exports = {
  steps: exec("ls test/*").split("\n").map((d) => new CommandStep(`run_test ${d}`))
}
```

---

`pipeline.js`

```javascript
const job = require("buildkite/job");

const data = gql`
  query {
    build(slug: "${job.build.pipeline.organization.slug}/${job.build.pipeline.slug}/${job.build.number - 1}") {
      metaData
    }
  }
`

module.exports = {
  steps: [
    {
      command: "echo ${data.build.metaData.somethingWasSetHere}"
    }
  ]
}
```

---

`command`

```javascript
#!/bin/bash
set -euo pipefail

if [[ "${BUILDKITE_PLUGIN_GOLANG_DEBUG:-false}" =~ (true|on|1) ]] ; then
  echo "--- :hammer: Enabling debug mode"
  env | sort | grep BUILDKITE_PLUGIN_GOLANG
fi

go_version="${BUILDKITE_PLUGIN_GOLANG_VERSION:-latest}"
import="${BUILDKITE_PLUGIN_GOLANG_IMPORT:-${BUILDKITE_PIPELINE_SLUG}}"

args=(
  "--rm"
  "--volume" "$PWD:/go/src/${import}"
  "--workdir" "/go/src/${import}"
)

# Parse extra env vars and add them to the docker args
while IFS='=' read -r name _ ; do
  if [[ $name =~ ^(BUILDKITE_PLUGIN_GOLANG_ENVIRONMENT_[0-9]+) ]] ; then
    args+=( "--env" "${!name}" )
  fi
done < <(env | sort)

echo "+++ :go: Running ${BUILDKITE_COMMAND} in golang:${go_version}"

docker run "${args[@]}" "golang:${go_version}" sh -c "${BUILDKITE_COMMAND}"
```

`command.sh`

```javascript
if (plugin.config.debug) {
  console.log("--- :hammer: Enabling debug mode");
  console.dir(plugin.config);
}

let goVersion = plugin.config.version;
if (goVersion == "") { goVersion = "latest" };

let importPackage = plugin.config.import;
if (importPackage == "") { importPackage = job.pipeline.slug };

let args = [ "--rm", "--volume" "$PWD:/go/src/" + importPackage, "--workdir" "/go/src/" + importPackage ];

if (plugin.config.env) {
  plugin.config.env.forEach((env) => {
    args.push("--env " + env);
  });
}

console.log("+++ :go: Running " + job.command + " in golang:" + golangVersion);

$`docker run ${...args} ${"golang" + goVersion} sh -c ${job.command}`
```
