# Community Project Guidelines

Before opening a pull request to add your project to Community-Projects.md,
we recommend making sure your project adheres to the guidelines below. These
ensure your project can be used easily by others. 

The following checklist helps you and reviewers consider some of the
steps that should be taken to make a project accessible to users. 

**Checklist**
* [ ] The project should be added to the listing in alphabetical order, following
the structure established by existing projects on the listing.
* [ ] The project should be descriptive to your target audience. There should be
a README that explains its purpose and how to use it, ideally with examples.
* [ ] The (Go) project should use tools such as `go fmt` and `go vet` to ensure
correct formatting and correct behaviour.
* [ ] The project should be in a usable state. Releases or tags are recommended, 
with versioning in particular following Go guidelines, e.g. `v1.0.1`, `v2.3.8`.
* [ ] Checklist for libraries
  * [ ] The library should be documented in places that users interact with it, such as
  exported types and functions. This ensures that the library can be viewed on
  [pkg.go.dev](https://pkg.go.dev) with adequate documentation.
  * [ ] The library should follow idiomatic Go conventions where possible. Naming
  conventions should be followed.