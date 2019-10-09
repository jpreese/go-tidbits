# go-tidbits

## 10/9/2019
- When running `go get` on a project that uses `ldflags`, they won't be set. The application will have blank data. (e.g. version, build).

- When running `go get` outside of the main module for an application (i.e. working directory is not in the modules path), `replace`/`exclude` directives do not apply. This can result in `go get` operations that fail to compile.