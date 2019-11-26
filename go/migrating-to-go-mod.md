# Migrating to go mod

GOLANG community is moving towards go mod command instead of using glide for dependency resolving.

## Using go mod

To use go mod command You must have go >= 1.11

1. Run `go mod init` to initialize module dependencies.
2. Run `go mod vendor` to create go.mod and go.sum
3. Use `go mod download` in Dockerfile to download dependencies