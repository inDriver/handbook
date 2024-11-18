# Go Version Policy

This policy describes constraints on the Go version used in your application.

## Glossary

Latest Go version at the moment of writing is `1.22.3` where, according to semver terminology:

* `major` version is `1`;
* `minor` version is `22`;
* `patch` version is `3`.


## Policy

* `patch` version must be greater than `0`. We can use the latest released minor version of Go, but only after the first patch is released;
* `minor` version must be greater than `${last minor version} - 5`. So if the last minor version is `22`, the application version should be greater than `17`, meaning Go version `1.18.1` is the minimum version.


## Rationale

This policy ensures that the new version is stable and does not contain critical bugs, while also keeping the application reasonably up-to-date with the latest version.

[Go releases](https://go.dev/doc/devel/release)
