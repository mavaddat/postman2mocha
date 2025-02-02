# Postman 2 Mocha

Convert Postman collections and environment files to scripts that run under Mocha / Chai.

## Status

At the moment only some features of Postman are implemented. Current limitations include:

* Only JSON response bodies are supported
* `pm.environment` can be used but there is no support for globals, variables or collection variables
* No support for `pm.setNextRequest`

API features that haven't been implemented are included in the generated code as-is so you can translate them manually.

## Setup

```shell
git clone https://github.com/AutomationSchool/postman2mocha.git
cd postman2mocha
npm install
```

## Usage

To create a mocha project from scratch with dependencies included:

```shell
node src project $OUTPUT_DIR -c $POSTMAN_COLLECTION_FILES -e $POSTMAN_ENVIRONMENT_FILES
```

After the project has been generated you can run it with:

```shell
cd $OUTPUT_DIR
npm install
npm start
```

This will run the generated collections with no environment file loaded. You can specify the environment to be used as follows:

```shell
env_name=<environment name> npm start
```

Where the environment name should match one of the available environment files in the `env` directory (without the `.env` extension). Whether an environment file is used or not, environment variables are also passed to the generated tests and are available through `pm.environment` etc.

You can also translate individual collection or environment files with `node src collection <source> <dest>` and `node src environment <source> <dest>`. More information is available with `node src --help`.

## Contributing

You can [find out a bit more about the design](ABOUT.md) on the about page.
If you have any requests, suggestions or other feedback you can get hold of us at [info@automationlabs.io](info@automationlabs.io).
