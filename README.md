# mentorpal.site (beanstalk app)

Use this repo to deploy the latest version (or any version) of mentorpal to your own mentorpal site as an [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) application.

## Deploying your site

The instructions below will guide you to set up an mentorpal site, running on `AWS Elastic Beanstalk` for your own domain name.

### Prerequisites

You need an `AWS` account with your Elastic Beanstalk infrastructure already in place. You can use our [terraform template](https://github.com/mentorpal/aws-beanstalk-terraform) to build and maintain this infrastructure.

### Required Software

- unix shell
- git
- git lfs
- curl

### Creating the github (fork-LIKE) repo for your site

The goal is to allow you to manage your deployment without having to edit any code or configuration, other than some environment vars/secrets for the [github actions](https://github.com/features/actions) in your own copy of this repo. So keep that in mind when reading the instructions below.

**Step 1. Create and clone a repo for your site**

Do this the normal way in your github.com account. If you like name your repo the same as the domain name of your mentorpal site + a suffix, e.g. `mentorpal.yourdomain.org-beanstalk-app`

You probably want this repo to be private, and you can use whatever option to make sure your repo has its `main` branch with a first commit (e.g. choose the option to create a README when you create the repo). Whatever commits you put in this repo will just be overwritten with tagged versions the mentorpal aws-beanstalk-app repo.

**Step 2. Initialize your repo as a fork-like clone mentorpal's [aws-beanstalk-app](https://github.com/mentorpal/aws-beanstalk-app.git) repo**

In a unix terminal, cd to the root of your repo clone and then execute:

```bash
curl -s -H "Accept:application/vnd.github.v3.raw" https://api.github.com/repos/mentorpal/aws-beanstalk-app/contents/install.sh | sh
```

This will change the `upstream` remote for your clone to the mentorpal source repo, set up git lfs etc.

The result will end up being a lot like a fork, but it can't be an actual fork or `github actions` will not run for you.

### Configure your repo to deploy to your site using [github secrets](https://docs.github.com/en/actions/reference/encrypted-secrets)**

Before you can deploy a version of mentorpal to your site, you need to configure your repo so github actions will deploy to the right account, instance, etc.

The goal is to allow you to use a fork-like clone of this repo to manage your deployment--including switching mentorpal release versions--without having to edit code/config or do any complicated git merging. To enabled this, all the details that specify your site and AWS account are stored as `github secrets`. We use `github secrets` for config whether it's really secret or not because secrets is the mechanism `github actions` provides for configuring enviroment variables that can be accessed in `github actions`.

You will need to configure all of the following secrets:

 - *AWS_ACCESS_KEY_ID*
 - *AWS_SECRET_ACCESS_KEY*
 - *AWS_REGION*
 - *EBS_APP_NAME*
 - *EBS_ENV_NAME_PROD*
 - *EFS_FILE_SYSTEM_ID*

If you don't know what any or all of these values should be, whoever setup your Elastic Beanstalk infrastructure in AWS should be able to configure them in github secrets for you, and/or see the [terraform template](https://github.com/mentorpal/aws-beanstalk-terraform)

### Deploying a version of mentorpal to your site domain

To publish a specific tagged release do:

```bash
./version publish <tag>
```

...to publish the latest version do:

```bash
./version publish --latest
```

The `./version publish` command will push a tag from upstream to your site's github repo. If the github config described above is complete and valid, this will trigger [github](https://github.com/features/actions) in your repo to deploy the tag to your configured AWS Elastic Beanstalk Env. You should see the deploy action running at `https://github.com/mentorpal/{YOUR_REPO}/actions`


### Switching your local clone to a released version of mentorpal

If you are developing on the app (as opposed to just deploying releases), first make sure you don't have any uncommitted/unstashed changes with `git status`

To switch the local-clone version to a release from [aws-beanstalk-app](https://github.com/mentorpal/aws-beanstalk-app) do:

```bash
./version switch <tag>
```

...to switch to the latest version do:

```bash
./version switch --latest
```


## Developing New Releases

To develop new releases, you will usually be working in a repo clone of a site that you own but pushing branches and changes to the upstream [aws-beanstalk-app](https://github.com/mentorpal/aws-beanstalk-app).

As an example, to set up a new branch from `upstream/main` you could follow these steps:

```bash
git fetch upstream
git checkout -b upstream/main my-new-branch
```

...then make changes and run the environment to test locally as describe above in this document.

When you are ready to publish your changes to your site to test...

1. push your branch to upstream, e.g.

```bash
git push upstream <my_branch>
```

2. On [aws-beanstalk-app](https://github.com/mentorpal/aws-beanstalk-app), create a release tag, e.g. `3.1.0-alpha.3`

3. back on your local machine, use the `version` tool to deploy your new release, e.g.

```bash
./version publish 3.1.0-alpha.3
```

## Setting up clone for local development and site deployment

1. Clone the site repository from github (e.g. careerfair.mentorpal.org-beanstalk-app)

2. In terminal from root, run command: 
```
sh ./install.sh
```

3. Check and make sure that the upstream remote has been set with command:
```
git remote -v
```
Output should look something like this:
```
origin	https://github.com/mentorpal/careerfair.mentorpal.org-beanstalk-app.git (fetch) <== your local repository
origin	https://github.com/mentorpal/careerfair.mentorpal.org-beanstalk-app.git (push)
upstream	https://github.com/mentorpal/aws-beanstalk-app.git (fetch) <== upstream release repository
upstream	https://github.com/mentorpal/aws-beanstalk-app.git (push)
```

4. (OPTIONAL) Create a local branch from the working upstream release branch
   - Use ``git branch -a`` to see all available branches
   - Make sure to create your release version branch from an **upstream** branch:
```
git checkout upstream/x.x.x-release -b x.x.x-release
```

5. (OPTIONAL) Running the app locally:

Access the .env secrets file from ICT 1password (need access) for your site

Copy the .env to env/test/.env

6. Creating a release
   - After committing and pushing changes, the changes will also be pushed to the upstream (check upstream from step 3)
   - Go to upstream git repository and create a new release tag (releases => draft a new release)
     - Give the tag a version number (x.x.x for final release or x.x.x-alpha.x for alpha release)
     - If alpha release, check the "This is a pre-release" checkbox
   - Publish the release
   - In local site repository, run ``./version publish <tag>`` from root

## FAQ

### Why can't I just fork this repo for my site?

We need a system where a github account can have multiple 'fork-like' copies of the mentorpal app deployment, one for each site domain.