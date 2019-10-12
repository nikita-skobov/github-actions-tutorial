# github-actions-tutorial
> a simple tutorial to reference when making github actions


# The goal

The goal is to be able to create actions that can automate a certain process given some inputs. These actions, if used often by many different projects, should get their own repository so that you do not need to copy the action code for every new repository that you want to use the action in.

After playing around with the new GitHub Actions Beta program, I figured out how to accomplish this.

For this example, let's consider two sample repositories: Fuzzy-Robot, and Fuzzy-Robot-Action

# Fuzzy-Robot-Action

This is the repository that will contain the action code that can be reused within various projects.

Create a new repository, and name it whatever you want (this example uses the name fuzzy-robot-action), add a README, and a license.

The first thing your action repo needs is an `action.yml` file in the root of the repo. This way, when you use this action in another repo, github knows where to look for your action metadata. This file being present also tells github that this repository is an action repository, and after it is created (and formatted properly) you will see a *Publish this Action to Marketplace* alert show up on the homepage of your github repo.

The action.yml simply contains the metadata for your action. You can find a [whole article here](https://help.github.com/en/articles/metadata-syntax-for-github-actions#runs) explaining the various options for this file, but here is a simple example:


```yml
name: 'Fuzzy Robot Action'
description: 'does a fuzzy robot action'
author: 'Nikita'
branding:
  icon: 'book'
  color: 'blue'
runs:
  # using: 'node12'
  # main: 'main.js'
  using: 'docker'
  image: 'Dockerfile'
```

Note that lines beginning with `#` are comments in yaml syntax.

I commented out the runs using node12 and main: main.js to show that that is an alternative option for defining an action. An action can either be ran using docker, or javascript. If it uses javascript, it runs faster because GitHub does not need to spin up a container for you. Also if using javascript, you simply specify a main entrypoint file. However, if you want to create a Docker action, you must specify an image. Note that this image can either be from your repository as we defined it here, or it can be a url to a public docker image.

Next, let's create a `Dockerfile` also in the root of the repo:

```sh
FROM node:12

LABEL "maintainer"="Nikita"


WORKDIR /usr/fra/temp

COPY entrypoint.sh /usr/fra/temp/

ENTRYPOINT ["sh", "/usr/fra/temp/entrypoint.sh"]
```

If you don't know much about Docker, it is best to look up a better guide for creating Docker images, but the gist of what we are doing here is that we create a file that describes what our docker container should start with.

We say `FROM node:12` because we want to use the already created `node` image with version 12. We give it a working directory and we copy a file `entrypoint.sh` into the working directory. Note that the `entrypoint.sh` file is a file in our repository that we will create shortly. We also tell Docker that when it runs this image, it should immediately run the `entrypoint.sh` file that we copied over into the docker image.

There are many more options that you can put in a Dockerfile. This guide will not go over the many features of Docker.

Next, let's create our `entrypoint.sh` file, also in the root of the directory:

```sh

#!/bin/sh

echo "this is the fuzzy robot action!"

echo "$SOME_PARAM"

env
```

We will explain why we used these 3 commands in the next section when we try to use our action:


# Fuzzy-Robot

Now that we have made our action repo, let's create a seperate repo that will actually use the action that we created. I am going to call it fuzzy-robot.

Once the repository is created, there is only one file that needs to be created to run your action. This file must be placed in the following directory: `.github/workflows/` for github to properly interpret it as an action that needs to be ran.

Let's call ours main.yml:

```yml
name: Main

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v1
    - name: res
      run: echo Hello, world!
    - name: test1
      uses: nikita-skobov/fuzzy-robot-action@master
      with:
        SOME_PARAM: ayyyyyyy
```

Here we create a yaml file and give it a name (which will show up in the github actions tab), an on property (which tells github when to run this action), and then we define our jobs.

The on property is important because in many cases we don't want to always run this action on every push. Sometimes we only want to run an action on a push to a specific branch, or maybe on a pull request, or a new issue, etc. Please see [this article](https://help.github.com/en/articles/workflow-syntax-for-github-actions#on) for more information on how to limit when your action is ran.

The important part here that I want to show is how you can specify another repository as an action via the `uses` keyword. It follows the format: `{owner}/{repo}/{path}@{ref}` 

Note that in this case our path is the root, so we do not specify a path. If we had multiple actions within a single `Fuzzy-Robot-Action` repository, we might want to seperate them by directory, and in that case we would use a path to differentiate them.

We also use `@master` to tell github which version/branch it is going to be using. using a branch is simplest, but you can also make a github release and specify a release number.

Lastly, we specify with: SOME_PARAM. This gets passed in as an environment variable to the action defined in fuzzy-robot-action.

Lets commit this file, which will automatically run the action, and let's see what the output is:

```
this is the fuzzy robot action!

ACTIONS_RUNTIME_TOKEN=***
NODE_VERSION=12.10.0
# ... more github environment variables
INPUT_SOME_PARAM=ayyyyyyy
GITHUB_BASE_REF=
# ... more github environment variables
```

The first line corrresponds to the first echo statement from our earlier `entrypoint.sh` file from the fuzzy-robot-action repository. The next line echoes $SOME_PARAM, which we provided in our `main.yml` file in the fuzzy-robot repository. However, we see that this line is blank. Why? Well it's because github prepends a INPUT_ to your environment variables. That is why we added the command `env` to our `entrypoint.sh` file so that we can see all of the environment variables that we have access to. There are a lot of them, so I ommited most of them, but you can see that somewhere in the middle is our `INPUT_SOME_PARAM` environment variable that contains the data that we passed to our fuzzy-robot-action.

I hope this gives a pretty good explanation of how to create a repository that can be used as a reusable/extendable action. Imagine you have a specific process such as minifying/compressing/deploying static assets to a website. Instead of copy/pasting the script that does this into every repository where you want to do this, you simply create a single action repository that contains these scripts, and then in your website repository you `use` this action repository, and simply pass in the relevant environment variables that are needed to deploy for that specific website. 

This is just one example of how this setup can be used. I hope this information was helpful.
