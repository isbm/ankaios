# CI/CD

As CI/CD environment Github Actions is used.
Merge verifications in case of opening a pull request and release builds are fully covered
into Github Action workflows. For information about release builds, see [CI/CD - Release](ci-cd-release.md) section.

## Merge verification

When a pull request is opened, the following pipeline jobs run:

- Linux-amd64 release build + tests in debug mode
- Linux-amd64 coverage test report
- Linux-arm64 release build (cross platform build)
- Requirements tracing

After a pull request was merged into the main branch, the jobs listed above
are executed again to validate stable branch behavior.

The steps for the build workflow are defined inside `.github/workflows/build.yml`.

The produced artifacts of the build workflow are uploaded and 
can be downloaded from Github for debugging or testing purposes.

## Adding a new merge verification job

To add a new merge verification job adjust the workflow defined inside `.github/workflows/build.yml`.

Select a Github runner image matching your purposes or in case of adding a cross-build first make sure that
the build works locally within the dev container.

1. Add a new build job under the `jobs` jobs section and define a job name.
2. Add the necessary steps to the job to build the artifact(s).
3. Append a use clause to the build steps to upload the artifacts to Github. If a new platform build is added name the artifact according to the naming convention `ankaios-<os>-<platform>-bin` (e.g. ankaios-linux-amd64-bin) otherwise define a custom name. If the artifact is needed inside a release the artifact is referenced with this name inside the release workflow.
   ```
    ...
     - uses: actions/upload-artifact@v3.1.2
       with:
         name: ankaios-<os>-<platform>-bin
         path: dist/
    ...
   ```

!!! note

    Github Actions only runs workflow definitions from main (default) branch.
    That means when a workflow has been changed and a PR has been created for that, the
    change will not become effective before the PR is merged in main branch.
    For local testing the [act](https://github.com/nektos/act) tool can be
    used.


## Adding a new Github action

When introducing a new github action, do not use a generic major version tag (e.g. `vX`).
Specify a specific release tag (e.g. `vX.Y.Z`) instead. Using the generic tag might lead to an unstable CI/CD environment,
whenever the authors of the Github action update the generic tag to point to a newer version that contains bugs or incompatibilities with the Ankaios project.

Example:

Bad:
```
...
  - uses: actions/checkout@v3
...
```

Good:
```
...
  - uses: actions/checkout@v3.5.3
...
```
