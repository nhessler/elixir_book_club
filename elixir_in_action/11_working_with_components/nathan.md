# 11 Working With Components

## 11.1

* `:application.which_applications` is a useful tool.
* Mix generates a plain text file with application info in it.
  * it has 3 interesting parts
    * `project/0` - defines project details
    * `application/0` - defines runtime dependencies and start options for main app
    * `deps/0` - defines compile time dependencies (called in `project/0`)
*`Application` is an OTP Behaviour
  * a a minimum you must define the `start/2` function
  * you cannot have multiple instances of an application running at the same time
  * Applications can be stopped with `Application.stop/1` with app name passed in
* Mix deps are a list of tuples {name, version, options}
* You can set the Mix environment by setting the OS Environment Variable `MIX_ENV`

## 11.2

* `mix deps.get` to get your dependencies
* `mix.lock` is similar to `Gemfile.lock`
* `mix` can detect and work with a `Makefile`
* gproc keys have 3 parts, {type, location, identifier}

## 11.3

* Plug has a sinatra like syntax if wanted
* Acts more like Rack though in that you pass the connection through the plugs
* BEAM's preemptive scheduler means long running requests don't keep everyone waiting
* you can override application config settings using the command line
