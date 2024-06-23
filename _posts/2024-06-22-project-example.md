---
title: "PDI project management"
date: 2024-06-22 00:00:00 -0800
categories: [pdi, pdi-project management]
tags: [pdi, pdi-project management]
---

## How to Manage PDI Projects
Compared to HOP, Pentaho Data Integration (PDI) does not offer built-in project or environment settings. As a result, you need to manage these settings manually. In this post, I'll show you how I handle my PDI projects and how you can set up your project environment to achieve maximum flexibility. You can download the blueprint (project-example) from my: [GitHub](https://github.com/dsanderbi/pdi-blog-post-samples)

### Project Folder Structure
Let's start with my preferred folder structure. In most cases, I use the structure shown below. Notice that in the `bin/` directory, separate folders represent different environments. Additionally, I store a separate `.kettle` folder for each environment in the `config` directory.

```
project-example/
├── bin/
│   ├── env-dev/
│   │   ├── env_settings.bat
│   │   └── env_settings.sh
│   ├── env-prod/
│   │   ├── env_settings.bat
│   │   └── env_settings.sh
│   ├── env-test/
│   │   ├── env_settings.bat
│   │   └── env_settings.sh
│   ├── start_pdi_gui.sh
│   ├── start_pdi_gui.bat
│   ├── project_settings.sh
│   └── project_settings.bat
├── config/
│   ├── dev/
│   │   └── .kettle/
│   ├── prod/
│   │   └── .kettle/
│   └── test/
│   │   └── .kettle/
├── logs/
├── main/
│   ├── jobs/
│   └── transformations/
├── tests/
│   ├── datasets/
│   ├── execution/
│   ├── metastore/
│   ├── test-results/
│   └── test-suites/
```

## Working with the Blueprint
To use the blueprint, you only need to define the environment variables in `env_settings.sh` (for Linux) or `env_settings.bat` (for Windows). This script sets the Java version for your PDI and the location of your PDI installation. It also allows you to set Pentaho Java options.

```bash
#!/bin/bash

# ************************************************
# ** Add your local configuration here          **
# ************************************************

# ************************************************
# **           SET PENTAHO_JAVA_HOME            ** 
# ************************************************
export PENTAHO_JAVA_HOME=
echo "- PENTAHO_JAVA_HOME=$PENTAHO_JAVA_HOME"

# ************************************************
# **           SET PATH TO DATA-INTEGRATION     ** 
# ************************************************
export PDI_HOME=
echo "- PDI_HOME=$PDI_HOME"

# ************************************************
# **           SET Xms || SET Xmx               ** 
# ************************************************
export PENTAHO_DI_JAVA_OPTIONS="-Xms4096m -Xmx8192m"
echo "- PENTAHO_DI_JAVA_OPTIONS=$PENTAHO_DI_JAVA_OPTIONS"
```
You need to set these options for all environments you want to use. Here, we have three environments:

1. **dev**: Used for local development. This file should be ignored by Git since each developer has different settings.
2. **test**: Used for ETL testing, typically on a Linux machine where the GitLab Runner tests the ETL.
3. **prod**: Used for production.


After setting the variables, you're ready to go by executing `start_pdi_gui.sh` (Linux) or `start_pdi_gui.bat` (Windows). This script opens the PDI GUI, allowing you to develop with PDI as usual.

### Setting Permissions
Before running the scripts, ensure you have set the appropriate permissions. On Linux, you need to make the scripts executable using the `chmod +x` command. Here’s how you can set the necessary permissions:

```bash
chmod +x bin/env-dev/env_settings.sh
chmod +x bin/env-prod/env_settings.sh
chmod +x bin/env-test/env_settings.sh
chmod +x bin/start_pdi_gui.sh
chmod +x bin/project_settings.sh
chmod +x bin/start_job.sh
```

Without setting these permissions, you might encounter issues when trying to execute the scripts. This step is crucial for ensuring that your environment setup and project management scripts run smoothly.
For Windows, ensure that your .bat files have the necessary execution permissions. Typically, this is not an issue since .bat files are executed by default, but you might need to run the Command Prompt or PowerShell as an administrator in some cases to avoid permission issues.

Let's examine what this script does:

```bash
#!/bin/bash

# Set the default environment to 'dev' if no parameter is passed
if [ -z "$1" ]; then
    env="dev"
else
    env="$1"
fi

# Check if the parameter is valid
if [[ "$env" != "dev" && "$env" != "test" && "$env" != "prod" ]]; then
    echo "Value not valid, use dev, test or prod!"
    exit 1
fi

echo "Start PDI-GUI with $env settings."
# ************************************************
# **           CALL PROJECT SETTINGS            ** 
# ************************************************
. ./project_settings.sh

# ************************************************
# **           CALL ENV SETTINGS                ** 
# ************************************************
. ./env-$env/env_settings.sh

# ************************************************
# **           CALL PDI-GUI                     ** 
# ************************************************
sh $PDI_HOME/spoon.sh $OPT "$@"
```
When starting the script without a parameter, the default environment loaded will be "dev". You can also open the PDI GUI on a production or test system by passing a parameter, if the system has a GUI. After that, the project settings will be set by the `project_settings.sh` script, which defines the `PROJECT_HOME` variable and adapts some project settings like the `logs` folder. Then, the environment settings are loaded.

```bash
#!/bin/bash

# ************************************************
# **           SET CURRENT_DIR                  ** 
# ************************************************
CURRENT_DIR=$(pwd)
echo "- CURRENT_DIR=$CURRENT_DIR"

# ************************************************
# **           SET PROJECT_HOME                 ** 
# ************************************************
export PROJECT_HOME=$(dirname "$CURRENT_DIR")
echo "- PROJECT_HOME=$PROJECT_HOME"

# ************************************************
# **           SET OPT                          ** 
# ************************************************
export OPT="$OPT -DPROJECT_HOME=$PROJECT_HOME"
echo "- OPT=$OPT"

# ************************************************
# **           SET KETTLE_HOME                  ** 
# ************************************************
export KETTLE_HOME="$PROJECT_HOME/config/$env"
echo "- KETTLE_HOME=$KETTLE_HOME"

# ************************************************
# **           SET TIMESTAMP                    ** 
# ************************************************
# Get the date in YYYYMMDD format
datestamp=$(date +%Y%m%d)
# Get the time in HHMMSS format
timestamp=$(date +%H%M%S)
# Combine date and time
TIMESTAMP="${datestamp}_${timestamp}"

# ************************************************
# **           SET LOGS_FOLDER                  ** 
# ************************************************
export LOGS_FOLDER="$PROJECT_HOME/logs"
mkdir -p "$LOGS_FOLDER"
mkdir -p "$LOGS_FOLDER/config_archive/config_$TIMESTAMP"

# ************************************************
# **           BACKUP CONFIGURATION             ** 
# ************************************************
cp -r "$PROJECT_HOME/config" "$LOGS_FOLDER/config_archive/config_$TIMESTAMP"
```
Each environment points to a different `.kettle folder`, allowing you to set different configuration parameters depending on the environment you are using. For example, you can set different values for variables or different database connections, providing great flexibility.

### Version Control Considerations
As mentioned above, development folders like `bin/env-dev/` and `config/dev` should be ignored by your `.gitignore` file since each developer may use different locations for their PDI. The test server configuration, however, typically remains the same for all developers and should be tracked, as should the production environment configuration.

### Running Jobs Without a GUI
Depending on the environment, you might not have the option or it might not be suitable to work with the PDI GUI. For this purpose, a wrapper script named `start_job.sh` has been created. This script allows you to start the `main.kjb` job included in the blueprint. By default, it starts `main.kjb` in the production environment with minimal logging output.
```bash
#!/bin/bash

# Check environment parameter
if [ -z "$1" ]; then
    env="prod"
else
    env="$1"
fi

# Check if the parameter is valid
if [[ "$env" != "dev" && "$env" != "test" && "$env" != "prod" ]]; then
    echo "Value not valid, use dev, test or prod!"
    exit 1
fi

# Check log level parameter
if [ -z "$2" ]; then
    LOG_LEVEL="Minimal"
else
    LOG_LEVEL="$2"
fi

echo "Start Job with $env settings and log level $LOG_LEVEL."

# ************************************************
# **           CALL PROJECT SETTINGS            ** 
# ************************************************
. ./project_settings.sh

# ************************************************
# **           CALL ENV SETTINGS                ** 
# ************************************************
. ./env-"$env"/env_settings.sh

# ************************************************
# **           CALL KITCHEN                     ** 
# ************************************************
"$PDI_HOME/kitchen.sh" -file="$PROJECT_HOME/main/jobs/main.kjb" -level="$LOG_LEVEL" "$OPT" >> "$PROJECT_HOME/logs/main_$TIMESTAMP.log" 2>&1
```

## Summary
Managing PDI projects effectively requires a structured approach to environment configuration. By setting up a clear folder structure and using environment-specific settings, you can ensure flexibility and consistency across development, testing, and production environments. This guide provides a blueprint for organizing your PDI projects, with scripts to streamline the setup and execution process. To download the blueprint, visit my [GitHub](https://github.com/dsanderbi/pdi-blog-post-samples).

I hope you find this post useful for managing your PDI projects. You are, of course, welcome to customize the blueprint to suit your needs and use it as a foundation for your projects.