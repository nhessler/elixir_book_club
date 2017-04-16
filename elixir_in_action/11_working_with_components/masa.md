# Elixir in Action - Chapter 11 - Working with components

## OTP application
- A reusable component
- A runtime OTP-specific construct
- A standard way of building Elixir/Erlang production systems and libraries.
- Can either:
  + run the entire supervision tree; or
  + just provide utility modules (as a library application)
- In a single BEAM instance, an application is a singleton.
- A non-library application is a callback module that must start the supervision tree.
- Application allows us to specify runtime dependencies to other applications.
- Application environments let clients tweak parameters of your applications without needing to change and recompile the application code.

### Application resource file
- A plain-text file written in Erlang terms that describes the application.
- We rely on the mix tool for it.
- contains:
  + The application names, version and description
  + A list of application modules
  + A list of application dependencies (which must be application themselves)
  + An optional application-callback module

### Creating OTP applications with mix tool

```bash
# Create a hello_world folder with minimum mix project skeleton.
# Make the application callback module and start the empty (childless supervisor) from it.
mix new hello_world --sup
```

```elixir
defmodule HelloWorld do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    # Define workers and child supervisors to be supervised
    children = [
      # Starts a worker by calling: HelloWorld.Worker.start_link(arg1, arg2, arg3)
      # worker(HelloWorld.Worker, [arg1, arg2, arg3]),
    ]

    opts = [strategy: :one_for_one, name: HelloWorld.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

```elixir
# Start the application in the iex.
cd hello_world
iex -S mix

# Check which application is currently running in the BEAM instance.
iex(1)> :application.which_applications
[{:hello_world, 'hello_world', '0.1.0'}, {:logger, 'logger', '1.3.4'},
 {:mix, 'mix', '1.3.4'}, {:iex, 'iex', '1.3.4'}, {:elixir, 'elixir', '1.3.4'},
 {:compiler, 'ERTS  CXC 138 10', '7.0.2'}, {:stdlib, 'ERTS  CXC 138 10', '3.1'},
 {:kernel, 'ERTS  CXC 138 10', '5.1'}]
```

```elixir
# The app is automatically started.
$ iex -S mix

# Only one instance per application is allowed.
iex(1)> Application.start(:hello_world)
{:error, {:already_started, :hello_world}}
iex(2)> Application.stop(:hello_world)
:ok
17:04:42.874 [info]  Application hello_world exited: :stopped
iex(3)> Application.start(:hello_world)
:ok
```

### Working with dependencies
- The main place where we specify an application is in the `mix.exs` file.
- Two types of dependencies:
  + build-time dependencies
    * We tell `mix` to fetch external code and compile it while building our project
  + runtime dependencies
    * We specify which application must be running in order for our application to work
- Erlang libraries, which rely on the `rebar` tool, can be used as dependencies
- `mix` can detect a `Makefile` in our `deps` and run `make` to build such projects

#### Configure dependencies

```elixir
# mix.exs
defmodule HelloWorld.Mixfile do
  use Mix.Project

  # Describe the mix project (compile-time construct)
  # Specify compile-time dependencies
  def project do
    [app: :hello_world,  # This atom is used to start/stop the application at runtime
     version: "0.1.0",
     elixir: "~> 1.3",
     build_embedded: Mix.env == :prod,
     start_permanent: Mix.env == :prod,
     deps: deps()]
  end

  # Describe the OTP application (runtime construct)
  # Specify runtime dependencies (a list of OTP applications that needed to be started before this application is started)
  def application do
    [applications: [:logger],
     mod: {HelloWorld, []}]
  end

  # If the OTP application is meant to be a library application, leave the runtime dependency list empty.
  # Library applications do not need their own supervision tree just for exposing utility modules.
  # def application do
  #   []
  # end

  # List dependencies
  defp deps do
    []
  end
end
```

#### Fetch dependencies

```elixir
# Fetch all dependencies recursively
mix deps.get
```

### The application behaviour
- An OTP behaviour
- Erlang's generic `:application` module
- Elixir's `Application` module, which provides some helper functions
- At minimum, must contain a `start/2` callback, which starts the top-level supervisor.


### The mix project environment
- A compile-time construct
- An atom that can be used to vary compile-time settings

```
# By informal convention, we specify a project environment by setting `MIX_ENV` OS environment variable
MIX_ENV=prod
```

### The compiled code structure
- `ebin/` - contains compiled binaries (`.beam` files and application resource files)
- `priv/` - contains application-specific private files (images, compiled C binaries, etc)

```
.
├── _build
│   ├── dev
│   │   ├── consolidated
│   │   │   ├── Elixir.Collectable.beam
│   │   │   ├── Elixir.Enumerable.beam
│   │   │   ├── Elixir.IEx.Info.beam
│   │   │   ├── Elixir.Inspect.beam
│   │   │   ├── Elixir.List.Chars.beam
│   │   │   └── Elixir.String.Chars.beam
│   │   └── lib
│   │       └── elixir_todo
│   │           └── ebin
│   │               ├── Elixir.Todo.Application.beam
│   │               :
│   │               └── elixir_todo.app
│   └── test
├── config
│   └── config.exs
├── lib
│   └── todo
│       ├── application.ex
│       :
│       └── system_supervisor.ex
├── mix.exs
:
```

---

## Web-server-related libraries

### Erlang
- [Cowboy](https://github.com/ninenines/cowboy)
  + Small, fast, modern HTTP server for Erlang/OTP. https://ninenines.eu

### Elixir
- [plug](https://github.com/elixir-lang/plug)
  + A specification and conveniences for composable modules between web applications
  + Similar to Ruby's Rack
  + Introduces the concept of stackable plugs:
    * middleware modules or functions that can be injected into the request-handling pipe
    * middleware modules or functions that intervene in various phases of request processing
- [Phoenix](http://www.phoenixframework.org/)
  + A productive web framework that does not compromise speed and maintainability.
- [httpoison](https://github.com/edgurgel/httpoison)
  + Yet Another HTTP client for Elixir powered by hackney

---

## Observer
- A graphical tool that ships with Erlang OTP
- Allows us to inspect running application and their supervision trees

```elixir
iex(1)> :observer.start
:ok
```

---

## call, cast or cast with queuing?
- First try to use calls
- Then switch to cast or casts with queuing depending on the specific situation

---

## Projects environment vs Application environment

### Project environment (mix environment)
- Allows us to tweak the outcome of compilation

### Application environments
- A key-value in-memory store
- Provides alternative values before the application is started
- Allows us to tweak the behavior of the system without having to recompile the code

#### In iex
- `iex` is an OTP application, whose runtime parameters can be configured via its environment

```elixir
iex(1)> Application.put_env(:iex, :default_prompt, "Elixir>")
:ok
Elixir>
```

#### Start the system using an Erlang option
- `--erl '-an_app_name key value'`

```
iex --erl '-iex default_prompt <<"Elixir>">>' -S mix
```

```
iex --erl "-todo port 5454" -S mix
```

#### List the default settings in `mix.exs`

```elixir
defmodule ElixirTodo.Mixfile do
  use Mix.Project
  # ...
  def application do
    [
      applications: [ :logger, :gproc ],
      mod: {Todo.Application, []},
      env: [
        port: 5454
      ]
    ]
  end
  # ...
end
```

#### List the default settings in `config/config.exs`

```elixir
use Mix.Config
# ...
config :todo, port: 5454
```
