# grpc-tools [![CircleCI](https://circleci.com/gh/bradleyjkemp/grpc-tools/tree/master.svg?style=svg)](https://circleci.com/gh/bradleyjkemp/grpc-tools/tree/master) ![GitHub tag (latest SemVer)](https://img.shields.io/github/tag/bradleyjkemp/grpc-tools.svg?label=version)

A suite of tools for gRPC debugging and development. Like [Fiddler](https://www.telerik.com/fiddler)/[Charles](https://www.charlesproxy.com/) but for gRPC!

The primary tool is `grpc-dump` which transparently intercepts network traffic and logs all gRPC and gRPC-Web requests as a JSON stream. You can then use this for this like debugging exactly what requests your application is making and exactly what responses the server returns.

![demo](demo.svg "Simple grpc-dump demo")

This repository currently includes:
* [`grpc-dump`](#grpc-dump): a small gRPC proxy that dumps RPC details to a file for debugging, and later analysis/replay.
* [`grpc-replay`](grpc-replay): takes the output from `grpc-dump` and replays requests to the server.
* [`grpc-fixture`](#grpc-fixture): a proxy that takes the output from `grpc-dump` and replays saved responses to client requests.
* [`grpc-proxy`](grpc-proxy): a library for writing gRPC intercepting proxies. `grpc-dump` and `grpc-fixture` are both built on top of this library.

These tools are in alpha so expect breaking changes between releases. See the [changelog](CHANGELOG.md) for full details.

### Installation:
The recommended way to install these tools is via [Homebrew](https://brew.sh/) using:
```bash
brew install bradleyjkemp/formulae/grpc-tools
```

Alternatively, binaries can be downloaded from the GitHub [releases page](https://github.com/bradleyjkemp/grpc-tools/releases/latest).

Or you can build the tools from source using:
```bash
go install github.com/bradleyjkemp/grpc-tools/...
```

## grpc-dump

Basic example:
```bash
# start the proxy (leave out the --port flag to automatically pick on)
grpc-dump --port=12345

# in another terminal, run your application pointing it at the proxy
HTTP_PROXY=localhost:12345 my-app

# all the requests made by the application will be logged to standard output in the grpc-dump window e.g.
# {"service": "echo", "method": "Hi", "messages": ["....."] }
# JSON will be logged to STDOUT and any info or warning messages will be logged to STDERR
```

Many applications expect to talk to a gRPC server over TLS. For this you need to use the `--key` and `--cert` flags to point `grpc-dump` to certificates valid for the domains your application connects to. The recommended way to generate these files is via the excellent [`mkcert`](https://github.com/FiloSottile/mkcert) tool:
```bash
# Configure your system to trust mkcert certificates
mkcert -install

# Generate certificates for domains you want to intercept connections to
mkcert mydomain.com *.mydomain.com

# Start grpc-dump using the key and certificate created by mkcert
grpc-dump --key=mydomain.com-key.pem --cert=mydomain.com.pem
```

More details for using `grpc-dump` can be found [here](grpc-dump/README.md).

## grpc-fixture

Basic example:
```bash
# save the (stdout) output of grpc-dump to a file
grpc-dump --port=12345 > my-app.dump

# in another, run your application pointing it at the proxy
HTTP_PROXY=localhost:12345 my-app

# now run grpc-fixture from the previously saved output
grpc-fixture --port=12345 --dump=my-app.dump

# when running the application again, all requests will
# be intercepted and answered with saved responses,
# no requests will be made to the real gRPC server.
HTTP_PROXY=localhost:12345 my-app
```

For applications that expect a TLS server, the same `--key` and `--cert` flags can be used as described above for `grpc-dump`.

More details for using `grpc-fixture` can be found [here](grpc-fixture/README.md).
