# Working with components

_OTP applications_ allow us to organize a system into reuseable components

## TLDR

- OTP applications are reusable components
- OTP applications are singletons in a single BEAM instance
- Non-library applications are callback modules that start a supervision tree
- Applications allow you to specify runtime dependencies to other applications
- Application environments let clients tweak parameters of your applications
  without changing/compiling the application code

## The Details

### 11.1.1 Describing an application

An _application resource file_ contains:
- application name, version, description
- list of application modules
- list of application dependencies
- application-callback module (optional)

It's kind of like npm's [`package.json`](https://docs.npmjs.com/files/package.json)

Example:
```elixir
defmodule MyProject.Mixfile do
  use Mix.Project

  def project do
    [app: :my_project,
     version: "0.0.1",
     elixir: "~> 1.0.0"
     deps: deps]
  end

  def application do
    [applications: [:logger],
     mod: {MyProject, []}]
  end

  def deps do
    []
  end
end
```

- Build time dependencies: `deps/0`
- Runtime dependencies: `application/0`

An application is a runtime construct,
but a `mix` project is a compile-time construct.

### 11.1.3 The `application` behavior

- An `application` is an OTP behaviour
- An `application` is a singleton in a BEAM instance
- At minimum, your callback module must contain a `start/2` function

### 11.1.4 Starting the application

If you want to start the system without the `iex` shell,
you can run `mix run --no-halt`.

### 11.1.5 Library applications

Library applications don't need an application callback module
because they don't need to create their own supervision tree.

### 11.1.7 The application folder structure

The project environment:
- A _project environment_ is a compile-time construct
- The `:only: :test` option ensures that the dependency
  is only used in the **test** project environment.
- You can specify the project environment by setting the `MIX_ENV`
  environment variable (i.e., `export MIX_ENV=test`)

The compiled code:
- Compiled binaries reside in `_build/ProjectEnv`

### 11.2.1 Specifying dependencies

- Dependencies are fetched from [Hex](https://hex.pm)
- Mix also works with Erlang libraries
- Mix can also detect Makefiles

### 11.3.1 Choosing dependencies

- [Phoenix](https://github.com/phoenixframework/phoenix)
- [Cowboy](https://github.com/ninenines/cowboy)
- [Plug](https://github.com/elixir-lang/plug)

### 11.3.2 Starting the server

- `:observer.start` is a GUI tool that ships with Erlang/OTP

### 11.4 Managing the application environment

- You can use `Application.put_env/3` to modify a parameter
- You can set default application configuration in `mix.exs`

```elixir
def application do
  [
    application [:gprod, :cowboy, :plug],
    mod: {Todo.Application, []},
    env: [
      port: 5454
    ]
  ]
end
```

- Another way to set configuration is through `config/config.exs`:

```elixir
use Mix.config
config :todo, port: 5454
```
