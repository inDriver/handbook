# Swagger

The Swagger documentation is mandatory for services.

We have added an OpenAPI check to the Go pipeline in the `base-workflow` repository.

This check performs the following steps:

1. **Disabled by default**: the check is off by default. To enable it, set the parameter `go_pipeline check_openapi_enabled` to `true`.

2. **Command execution**: it runs the command to generate the OpenAPI specification / code. By default, it runs `make generate-open-api`, but you can override this with the parameter `check_generate_open_api_cmd`. Note that only `go` is installed on the runner.

3. **Modified files check**: after running the command, the pipeline checks if there are any modified files in the repository. If modified files are found, it means the OpenAPI spec or code wasn't updated, and the pipeline will fail.

4. **`.documentary.toml` check**: the pipeline verifies that a `.documentary.toml` file exists and contains an `apispec` section where a `.json` or `.yaml` file is specified. If these conditions are not met, the pipeline will fail.

We recommend that all applications with an OpenAPI specification enable this check and configure it properly to ensure it passes successfully.