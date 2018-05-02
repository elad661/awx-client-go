# awx-client-go

[![Build Status](https://travis-ci.org/moolitayer/awx-client-go.svg?branch=master)](https://travis-ci.org/moolitayer/awx-client-go)

A golang client library for [AWX](https://github.com/ansible/awx) and [Ansible Tower](https://www.ansible.com/products/tower) REST API.

## Installation
Install awx-client-go using the "go-get" command:
```
go get github.com/moolitayer/golang/glog # Dependency
go get github.com/moolitayer/awx-client-go/awx
```

## Usage
### import
```
import 	"github.com/moolitayer/awx-client-go/awx"
```

### Creating a connection:
```
# Uses the builder pattern:
connection, err := awx.NewConnectionBuilder().
  Url("http://awx.example.com/api").
  Username(username).
  Password(password).
  Token("TOKEN").
  Bearer("BEARER").
  CAFile("/etc/pki/tls/cert.pem").
  Insecure(insecure).
  Proxy(http://myproxy.example.com).
  Build()                                      # Create the client
if err != nil {
  panic(err)
}
defer connection.Close()                      # Don't forget to close the connection!
```
`Url()` points at an AWX server's root API endpoint.
`Proxy()` specifies A proxy server to use for all outgoing connection to the AWX server.
#### Authentication
`Username()` and `Password()` specify Basic Auth for AWX API server.
`Token()` uses the authtoken/ endpoint and works with AWX XXX and Ansible tower < 3.3.
`Bearer()` uses OAuth2 and works since AWX XXX and Ansible Tower > 3.3.

#### TLS
`CAFile()` specifies a trusted PEM encoded CA certificates used to verify the AWX server.
`CAFile()` can be used multiple times to specify a list of files.
`Insecure(true)` can be specified to ignore TLS verification.

### Supported resources
- Projects
- Jobs
- Job Templates

### Retrieving resources
```
projectsResource := connection.Projects()

// Get a list of all Projects.
getProjectsRequest := projectsResource.Get()
getProjectsResponse, err := getProjectsRequest.Send()
if err != nil {
  panic(err)
}

// Print the results:
projects := getProjectsResponse.Results()
for _, project := range projects {
  fmt.Printf("%d: %s - %s\n", project.Id(), project.Name(), project.SCMURL())
}
```
#### Filtering
User `Filter()` on a request to filter lists:
```
projectsResource := connection.Projects()

// Get a list of all Projects using git SCM.
getProjectsResponse, err := getProjectsRequest.Filter("scm_type", "git").Send()

```
#### Retrieving resource by id
Use `id()` on a resource list to get a single resource
```
projectsResource := connection.Projects()

// Get project with id=4
getProjectsRequest := projectsResource.Get()
projectResource := connection.Projects().Id(4)

// Send the request to retrieve the project:
getProjectResponse, err := projectResource.Get().Send()
```

#### Launching a Job from a Template
```
// Launch Job Template with id=8
connection.JobTemplates().Id(8).Launch()
launchResource := connection.JobTemplates().Id(templateId).Launch()
response, err := launchResource.Post().
  ExtraVars("{\"awx_environment\": \"staging\"}").
  Limit("master.example.com").
  Send()
if err != nil {
  return err
}
```
`ExtraVars()` Specify a JSON objects of extra variable to pass with the job templates
`Limit()` is an Ansible host pattern.
See [Job Template](http://docs.ansible.com/ansible-tower/latest/html/userguide/job_templates.html)

## Examples

See [examples](examples).
