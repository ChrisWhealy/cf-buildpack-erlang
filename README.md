## Cloud Foundry Build Pack: Erlang

This is an updated Cloud Foundry build pack for Erlang OTP apps. Apps are built using [Rebar3](http://www.rebar3.org/) which in turn, assumes your Erlang app is a standard OTP app.


## Update

The following updates have been made:

* Erlang tarballs are downloaded from Heroku Cedar 14 stack
* Rebar3 is used instead of Rebar2.x
* Default Erlang OTP version is now `20.1`, not `master`
* Config variables can be exported to the build environment

### Export config variables to the build environment

Names of the config variables to be exported during build time are obtained from a dot file called `.env-export`. 
Names should be divided by the new line separator. If file is missing or empty no variables are exported.

To make config variable f.e. REBAR_PROFILE available during the compilation do the following.
Add the variable with `heroku config:set REBAR_PROFILE=prod`. Then add the `REBAR_PROFILE` string to the `.env-export` file.

### Select an Erlang version

The Erlang/OTP release version that will be used to build and run your application is now obtained from a dot file called `.preferred_otp_version`.  If this file is missing, the OTP version will default to `20.1`.

If this file can be found in the root directory of your repo, it must contain only the OTP version number you require.  Pre-compiled binaries from version from `17.0` upwards are available from the Cedar 14 stack.

#### Example

If your `.preferred_otp_version` file contains `19.2`, then the file `OTP-19.2.tar.gz` will be downloaded from `https://s3.amazonaws.com/heroku-buildpack-elixir/erlang/cedar-14/`.

### Create an Erlang Release

Please ensure that the `rebar.config` file of your OTP app contains a release definition.  Something like:

    {relx, [
        {release,
          { <app_name>, "1.0.0" },
          [ <app_name>, sasl, runtime_tools]
        },

        {sys_config, "./config/sys.config"},
        {vm_args,    "./config/vm.args"},

        {dev_mode, false},
        {include_erts, true}
      ]
    }.

***IMPORTANT***  
In your `vm.args` file, add the `+B` flag to disable the break handler.  This then means that your app can be terminated with a single Ctrl-C, instead of the normal Ctrl-C followed by another Ctrl-C.

### Create a Procfile for CF to run

Erlang releases are run via their application name, so in the root directory of your repo, create a `Procfile` containing the following:

    web: _build/default/rel/<app_name>/bin/<app_name> foreground

***IMPORTANT***  
You must use the `foreground` command to start your app, not the `start` command.  The `start` command returns control to the OS which makes Cloud Foundry think your app has terminated with exit code 0.  This causes a startup error saying `Codependent process exited` and even though there's nothing wrong with your app, Cloud Foundry will not be able to run it.


### Build your CF App

    $ cf push <app_name> -b https://github.com/ChrisWhealy/cf-buildpack-erlang
  
