[![PkgGoDev](https://pkg.go.dev/badge/github.com/hashicorp/terraform-exec)](https://pkg.go.dev/github.com/hashicorp/terraform-exec)

# terraform-exec fork

This fork of the [terraform-exec](https://github.com/hashicorp/terraform-exec) library was necessary to be able to run Terraform inside a lambda function. Even though this is not a recommended approach, it is useful in the case that you want the lambda function to clone an existing repository and run Terraform on it.

## Go compatibility

This library is built in Go, and uses the [support policy](https://golang.org/doc/devel/release.html#policy) of Go as its support policy. The two latest major releases of Go are supported by terraform-exec.

Currently, that means Go **1.18** or later must be used.

## Usage

The `Terraform` struct must be initialised with `NewTerraform(workingDir, execPath)`. 

Top-level Terraform commands each have their own function, which will return either `error` or `(T, error)`, where `T` is a `terraform-json` type.


### Example


```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/hashicorp/go-version"
	"github.com/hashicorp/hc-install/product"
	"github.com/hashicorp/hc-install/releases"
	"github.com/hashicorp/terraform-exec/tfexec"
)

func main() {
	installer := &releases.ExactVersion{
		Product: product.Terraform,
		Version: version.Must(version.NewVersion("1.0.6")),
	}

	execPath, err := installer.Install(context.Background())
	if err != nil {
		log.Fatalf("error installing Terraform: %s", err)
	}

	workingDir := "/path/to/working/dir"
	tf, err := tfexec.NewTerraform(workingDir, execPath)
	if err != nil {
		log.Fatalf("error running NewTerraform: %s", err)
	}

	err = tf.Init(context.Background(), tfexec.Upgrade(true))
	if err != nil {
		log.Fatalf("error running Init: %s", err)
	}

	state, err := tf.Show(context.Background())
	if err != nil {
		log.Fatalf("error running Show: %s", err)
	}

	fmt.Println(state.FormatVersion) // "0.1"
}
```

## Testing Terraform binaries

The terraform-exec test suite contains end-to-end tests which run realistic workflows against a real Terraform binary using `tfexec.Terraform{}`.

To run these tests with a local Terraform binary, set the environment variable `TFEXEC_E2ETEST_TERRAFORM_PATH` to its path and run:
```sh
go test -timeout=20m ./tfexec/internal/e2etest
```

For more information on terraform-exec's test suite, please see Contributing below.

## Contributing

Please see [CONTRIBUTING.md](./CONTRIBUTING.md).
