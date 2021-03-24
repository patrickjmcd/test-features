# Sample Automated Feature Publishing

A POC to test automatically generating HTML files from feature files and sending the HTML to Slack

## Ok, that sounds great, but why?

Feature files are awesome. They're an excellent way to mesh business requirements with softare testing. One problem that exists, however, is that feature files are typically stored in github as something like `docs/features/thing/01-ABCD.feature`. Business users typically don't want to muck around in Github, and they often don't even have credentials to access the code.

You *could* create a separate repository solely for feature files to isolate the business users to their own repo, but there are plenty of headaches with that approach. To access the feature files in the tests, you'll need to add the feature files repo as a submodule of your application repo and create some hooks to make sure they're always up to date. You'll also need to train your business users on using Github. I've found that once someone at a certain level sees `# commented out text`, even in a feature file, they think they're doing something wrong.

My proposed solution to this problem is to generate an HTML file of all the features and upload it to a Slack channel. This keeps the business users out of Github and the code and yet, gives them visibility to the files in a clean, formatted fashion. Most users are familiar with HTML files, especially a nicely formatted one!

## Requirements

* A github repository with some `.feature` files
* Include my `features2html` npm package in your `devDependencies` in your `package.json`
* Create a script in your `package.json` file that runs the `features2html` command (see this repo's package.json for an example)
* A slack application with `files.upload` permissions ([see here](https://api.slack.com/authentication/basics) for how to set up the app)
* Github repository secrets set up for `SLACK_TOKEN` from your bot and `SLACK_CHANNEL`, a 

## How it works

Check out the `.github/workflows/main.yml` file:

```YAML
name: Feature File CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: 15.x
    - run: npm install
    - run: npm run features2html:ci
    - uses: MeilCli/slack-upload-file@v1
      id: message
      with:
        slack_token: ${{ secrets.SLACK_TOKEN }}
        channels: ${{ secrets.SLACK_CHANNELS }}
        file_path: '/tmp/features.html'
        file_name: 'features.html'
        file_type: 'html'
        initial_comment: 'feature file post by slack-upload-file'
    - run: 'echo ${{ fromJson(steps.message.outputs.response).file.permalink }}'

```

This Github action runs on a ubuntu environment:

1. The repository is checked out
2. A node environment is setup (this example uses node version 15.x)
3. The action runs `npm install` to pull in dependencies
4. The action then runs `npm run features2html:ci`, the custom script for generating the html file
5. The outputted HTML file is then uploaded to the specified Slack channel
6. The action prints out a permalink to the file


## References

* [features2html](https://github.com/patrickjmcd/features2html)
* [Github Action: slack-upload-file](https://github.com/marketplace/actions/slack-upload-file)