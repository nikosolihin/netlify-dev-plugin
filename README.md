# netlify-dev-plugin

Netlify CLI plugin for local dev experience.

## What is Netlify Dev?

Netlify Dev brings the power of Netlify's Edge Logic layer, [serverless functions](#netlify-functions) and [add-on ecosystem](#using-add-ons) to your local machine. It runs Netlify's production routing engine in a local dev server to make all redirects, proxy rules, function routes or add-on routes available locally and injects the correct environment variables from your site environment, installed add-ons or your netlify.toml file into your build and function environment.

It automatically detects common tools like Gatsby, Hugo, React Static, Eleventy, and more, to give a zero config setup for your local dev server and can help scaffolding new functions as you work on them.

## Prerequisites

- You should have the latest Netlify CLI version (`netlify -v` gives v2.10 and up) -`npm install -g netlify-cli` if not
- You should be [logged in on Netlify CLI](https://www.netlify.com/docs/cli/#authentication)
- Your project should be linked to a `siteID` on Netlify (using [netlify init](https://www.netlify.com/docs/cli/#continuous-deployment) or [netlify link](https://www.netlify.com/docs/cli/#linking-and-unlinking-sites)). You can confirm this has been done if you have a `.netlify` folder with a `state.json` file containing your `siteID`.

This is how we pull down your build environment variables and manage your addons on your local machine.

## Usage

- `netlify dev` start a local dev server for the build tool you're using
- `netlify dev:exec <command>` runs a shell command within the netlify dev environment
- `netlify functions:create` bootstrap a new function

As these commands are expected to be frequently used, it may be helpful to define aliases in your terminal (Mac: [bash](https://jonsuh.com/blog/bash-command-line-shortcuts/), [zsh](https://askubuntu.com/questions/758496/how-to-make-a-permanent-alias-in-oh-my-zsh), Windows: [doskey](https://stackoverflow.com/questions/20530996/aliases-in-windows-command-prompt), [registry](https://stackoverflow.com/questions/20530996/aliases-in-windows-command-prompt)) to your personal preference. For example:

```bash
## ~/.zshrc
alias ndeploy="netlify deploy --prod"
alias nd="netlify dev"
alias ndl="netlify dev --live"
alias nfc="netlify functions:create"
alias ndx="netlify dev:exec "
```

## Using the beta

Currently the Netlify dev plugin is in private beta. You'll need to follow these steps to enable it:

Make sure Netlify CLI is installed and up to date:

```bash
npm install -g netlify-cli
```

Then clone and activate the plugin:

```bash
git clone git@github.com:netlify/netlify-dev-plugin.git
cd netlify-dev-plugin
npm install
netlify plugins:link .
```

Now you're both ready to start testing netlify dev and to contribute to the project.

### Netlify Dev usage

```bash
USAGE
  $ netlify dev

OPTIONS
  -c, --cmd=cmd              command to run
  -f, --functions=functions  Specify a functions folder to serve
  -o, --offline              disables any features that require network access
  -p, --port=port            Specify port of netlify dev
  -l, --live                 Start a public live session

DESCRIPTION
  The dev command will run a local dev server with Netlify's proxy and redirect rules

EXAMPLES
  $ netlify dev
  $ netlify dev -c "yarn start"
  $ netlify dev -c hugo

COMMANDS
  dev:exec  Exec command
```

### Redirects

Netlify Dev has the ability emulate the [redirect capability](https://www.netlify.com/docs/redirects/) Netlify provide on the [ADN](https://netlify.com/features/adn) in your local environment. The same redirect rules which you configure to run on the edge, will also work in your local builds.

Netlify dev supports redirect rules defined in either `_redirects` or `netlify.toml` files.

The order of precedence for applying redirect rules is:

1. `_redirects` file (in the project's publish folder)
1. `netlify.toml` file (in the project's publish folder)
1. `netlify.toml` file (in the project's root folder)

See the [Redirects Documentation](https://www.netlify.com/docs/redirects/) for more information on Netlify's redirect and proxying capabilities.

#### Running the project and accessing redirects

```bash
# Build, serve and hot-reload changes
$ netlify dev
```

### Project detection

Netlify Dev will attempt to detect the SSG or build command that you are using, and run these on your behalf, while adding other development utilities.

**Overriding the detectors**: The number of project types which Netlify Dev can detect is growing, but if yours is not yet supported automatically, you can instruct Netlify Dev to run the project on your behalf by declaring it in a `[dev]` block of your `netlify.toml` file.

```toml

#sample dev block in the toml
[dev]
  command = "yarn start" # Command to start your dev server
  port = 3000 # Port that the dev server will be listening on
  publish = "dist" # Folder with the static content for _redirect file
```

**Explanation of detectors and ports in Netlify Dev**: There will be a number of ports that you will encounter when using Netlify Dev, especially when runnign a static site generator like Gatsby which has its own dev server. All the port numbers can be a bit confusing, so here is a brief explainer. If your SSG has a devserver on port 8000 for example, Netlify Dev needs to be told to proxy that port so it can merge it in with the rest of the local Netlify environment (say, running on port 8888), which is what you want to get the full Netlify Dev experience with Functions, Redirects, and so on. If you're running a project we have a detector for, we hardcode those conventional ports so you don't have to supply it yourself. However, sometimes you're using some other project (we welcome contributions for detectors!) or just have a custom port you want Netlify Dev to point to for some reason. This is when you go to the `netlify.toml` `[dev]` block to specify exactly what port we should listen to.

As for which port to use while doing local development in Netlify Dev, always look for this box in your console output and use that:

```bash
   ┌──────────────────────────────────────────────────────────────┐
   │                                                              │
   │   [Netlify Dev] Server now ready on http://localhost:64939   │
   │                                                              │
   └──────────────────────────────────────────────────────────────┘
```

### Netlify Functions

Netlify can also create serverless functions for you locally as part of Netlify Functions. The serverless functions can then be run by Netlify Dev in the same way that wold be when deployed to the cloud.

A number of function templates are available to get you started, and you can add your own utility functions to suit your own project development needs.

Create a new function

```bash
$ netlify functions:create
```

More detailed usage examples:

```bash
# Create a new function from one of the
# available templates offered when prompted (see below)
$ netlify functions:create

# alternatives
$ netlify functions:create hello-world # Create a new function with a given name
$ netlify functions:create --name hello-world # same

# Create a new function by cloning a template from a remote url
# organised with dependencies installed into a subdirectory
$ netlify functions:create hello-world --url https://github.com/netlify-labs/all-the-functions/tree/master/functions/9-using-middleware
```

**Function Templates**

Function templates can specify `addons` that they rely on as well as execute arbitrary code after installation in an `onComplete` hook, if a special `.netlify-function-template.js` file exists in the directory:

```js
// .netlify-function-template.js
module.exports = {
  addons: ["fauna"],
  onComplete() {
    console.log(`custom-template function created from template!`);
  }
};
```

#### Executing Netlify Functions

After creating serverless functions, Netlify Dev can serve them to you as part of your local build. This emulates the behaviour of Netlify Functions when deployed to Netlify.

```bash
# Build, serve and hot-reload changes
$ netlify dev
```

Each serverless function will be exposed on a URL corresponding to its path and file name.

`./functions/hello-world.js` -> `http://localhost:{PORT}/.netlify/functions/hello-world`

`./functions/my-api/hello-world.js` -> `http://localhost:{PORT}/.netlify/functions/my-api/hello-world`

### Using Add-ons

Add-ons are a way for Netlify users to extend the functionality of their Jamstack site/app.

[Add-on docs](https://www.netlify.com/docs/partner-add-ons/).

To try out an add-on with Netlify dev, run the `netlify addons:create` command:

```bash
netlify addons:create fauna
```

The above command will install the FaunaDB add-on and provision a noSQL database for your site to leverage. The FaunaDB add-on injects environment variables into your site's build process and the serverless functions.

Or install this [one click example](https://github.com/netlify/fauna-one-click).

After you have installed an add-on, it will be visible with the `netlify addons:list` command inside your site's current working directory.

## Live Share

To share your ongoing dev session with a coworker, just run Netlify Dev with a `--live` flag:

```bash
netlify dev --live
```

You will get a URL that looks like `https://clever-cray-2aa156-6639f3.netlify.live/`. This can be accessed by anyone as long as you keep your session open.

> Note: there are currently known issues with ending the live session alongside your webdevserver. We are working on fixing it. In the mean time you can run `ps aux | grep live-tunnel` and kill these sessions manually.
