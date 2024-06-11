:confused: **[Multi-stage](https://docs.docker.com/build/building/multi-stage/) Build?**

- Reduce image size.
- Multiple `FROM` statements, each can use a different base, and each of them begins a new stage of the build.

```dockerfile
# syntax=docker/dockerfile:1

# stage#0: build a bin in base image golang, alias build
FROM golang:1.21 as build
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

# stage#1: bin copied from 1st stage to next in base image scratch
# --from=build, alias to stage#0
FROM scratch
# COPY --from=0 /bin/hello /bin/hello
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

```bash
$ docker build -t hello .
$ docker images | grep hello  # only 1.8M!
```

- [dive](https://github.com/wagoodman/dive)

```bash
$ dive hello
```