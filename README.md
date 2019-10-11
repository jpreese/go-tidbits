# go-tidbits

## 10/9/2019
- When running `go get` on a project that uses `ldflags`, they won't be set. The application will have blank data. (e.g. version, build). Something to consider when using this functionality and suggesting users to get your project via `go get` instead of providing a binary produced through your build system.

- When running `go get` outside of the main module for an application (i.e. working directory is not in the modules path), `replace`/`exclude` directives do not apply. This can result in `go get` operations that fail to compile if the module takes the liberty of replacing and excluding certain dependencies.

## 10/10/2019
- Go 1.13 introduced the `-trimpath` argument which can assist in creating reproducible builds. When an error occurs, the stack trace has information relating to the build environment, this means a binary can be different based on where its built from. `-trimpath` removes these paths and only uses the module path.