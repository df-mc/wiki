This guide on getting started with Dragonfly development assumes a basic
understanding of Go (and Git). If you do not have experience with Go yet,
it is recommended to go through the [Go Tour](https://go.dev/tour/welcome/1)
and/or other resources.

## Index
* [Starting your project](#starting-your-project)
* [Understanding the `main.go` file](#understanding-the-maingo-file)
* [Using libraries](#using-libraries)

## Starting your project
Dragonfly by itself is a library that can be imported into your code. It
is generally easy to use the [Dragonfly template](https://github.com/df-mc/template)
to create a new repository with the basics necessary to run a Dragonfly
server. After clicking `Use this template` and creating your repository,
you can run `git clone` to clone the repository locally.

To run Dragonfly, navigate into the folder `git clone` created and run:
```shell
go run main.go
```

Dragonfly should now start and display log messages that look like this:
```
2024/11/23 12:57:23 INFO Opened dimension. dimension=overworld name=World
2024/11/23 12:57:23 INFO Opened dimension. dimension=nether name=World
2024/11/23 12:57:23 INFO Opened dimension. dimension=end name=World
2024/11/23 12:57:23 INFO Starting Dragonfly server... mc-version=1.21.40 go-version="go1.23.3" commit=2242b772bcc4f65d4f8e12cfab8cf70c2d78cb1a
2024/11/23 12:57:23 INFO Server running. addr=[::]:19132
```

Your server is now running. By default, Dragonfly will run on port 19132.
This setting and others may be edited in the config.toml file.

To stop the server, `ctrl+C` can be run. The server will then shut down
gracefully.

## Understanding the `main.go` file
Now that we are able to run the server, it is time to have a look at the
[`main.go` file](https://github.com/df-mc/template/blob/master/main.go).

The file starts with the package declaration and imports:
```go
package main

import (
	"fmt"
	"github.com/df-mc/dragonfly/server"
	"github.com/df-mc/dragonfly/server/player/chat"
	"github.com/pelletier/go-toml"
	"github.com/sirupsen/logrus"
	"io/ioutil"
	"os"
)
```
Your executable file must always be `package main` so that Go can compile
it into a standalone executable. The imports are used in the code following
them.

Following that, we reach the `main` function declaration. `main` functions
are unique to files that have `package main`. This function is called when
the compiled executable is run.
```go
func main() {
    chat.Global.Subscribe(chat.StdoutSubscriber{})
```
The first thing done here is the creation of a 
`chat.StdoutSubscriber{}`. This is a standard implementation of the 
`chat.Subscriber` interface that prints chat messages to the console when 
subscribed to a chat.

After that, the `config.toml` file is read. This calls the `readConfig`
function declared further down in the `main.go` file, which uses the 
`pelletier/go-toml` library to read a `server.Config`:
```go
config, err := readConfig(slog.Default())
if err != nil {
    log.Fatalln(err)
}
```
Additionally, a logger is passed that will be used by the server to log
information.

The `server.Config` read from the `config.toml` and logger are then used 
to create a new server:
```go
srv := conf.New()
srv.CloseOnProgramEnd()
```
The `srv.CloseOnProgramEnd()` call ensures the server shuts down gracefully
when the program is stopped using `ctrl+C`. After this, the server is started.

The last lines of this `main` function are used to start listening and accept 
players joining the server:
```go
srv.Listen()
for p := range srv.Accept() {
	_ = p
}
```
This snippet is generally the first place you will want to edit when you
start working on your own server. `srv.Accept()` returns players
as soon as they join the server.

You are now able to listen for a player joining and run code when they
do. Further developer resources will go into depth on how to attach a
handler to a `*player.Player` to listen for events.

## Using libraries
Because Dragonfly is just a library that is being used in this program,
it is trivial to import external libraries and use them. You can simply
run `go get github.com/<user>/<repo>` to add a library. Go will then
add the library to the `go.mod` and `go.sum` files, and you will be able
to use them normally.