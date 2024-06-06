---
title: "First Hop project"
date: 2024-06-06 00:00:00 -0800
categories: [hop, hop-setup]
tags: [hop, hop-setup]
---

## Set up HOP and Create Your First Project

I want to share how to set up your first project and configure environment options in HOP.

### Download and Set up

1. **Install HOP**: Ensure you have Apache HOP installed. You can download the latest version from the [official website](https://hop.apache.org/download/).

2. **Start HOP-GUI**: Unzip the file. But before you launch HOP by 

```bash
# Linux
sh hop-gui.sh
```
```batch
REM Windows
.\hop-gui.bat
```
set the "HOP_CONFIG_FOLDER" variable. This allows you to keep you configuration, projects and enviroments independent from your installed HOP version.

To set the "HOP_CONFIG_FOLDER" edit your enviroment variables:

![env var](assets/img/blogpics/blog-post-2/p2.png)

Click on "Enviroment Variables".

![env var2](assets/img/blogpics/blog-post-2/p3.png)

Select "Edit" and include "HOP_CONFIG_FOLDER" with the path you like to store your configurations.

![env var2](assets/img/blogpics/blog-post-2/p4.png)

Now copy the "hop-config.json" file stored in "hop\config" to your folder defined in "HOP_CONFIG_FOLDER". 
This enures that you have the default projects delivered by HOP.

> This setting must not be done when you start out with HOP. You can add the "HOP_CONFIG_FOLDER" later as well.
{: .prompt-tip }

If you use Linux this is also straigt forward. Just add the variable to your **bashrc** file.

```bash
# open file with nano
nano ~/.bashrc
# Add this to the end of the file
export HOP_CONFIG_FOLDER=/home/user/<your_folder>
# save the file and execute 
source ~/.bashrc
```

## First Project

1. **Launch HOP**: Open the HOP GUI & create your first project!

![create project](assets/img/blogpics/blog-post-2/p5.png)

![create project](assets/img/blogpics/blog-post-2/p6.png)

Add a folder if needed otherwise select a existing one for your project.

![create project](assets/img/blogpics/blog-post-2/p7.png)



4. **Create a New Project**:
    - Navigate to the **File** menu and select **New** > **Project**.
    - Enter a name for your project and choose a location where it will be stored.
    - Click **Create** to initialize the project.

### Configuring Environment Options

HOP environments allow you to manage different configurations and settings for various stages of your project (e.g., development, testing, production).

1. **Define Environments**:
    - In the HOP GUI, go to **Project** > **Environments**.
    - Click **Add** to create a new environment.
    - Provide a name for your environment (e.g., Development, Testing, Production).
    - Specify the necessary configurations, such as variables, parameters, and connection details.

2. **Set Up Environment Variables**:
    - Within the environment settings, define any environment-specific variables.
    - These variables can include database connection strings, file paths, or other settings that vary between environments.

3. **Switch Between Environments**:
    - To switch environments, go to **Project** > **Environments** and select the desired environment.
    - This will load the specific settings and configurations for the selected environment.

By setting up environments in HOP, you can easily manage different configurations for various stages of your project lifecycle, ensuring a smooth transition from development to production.

### Getting Started with Workflows & Transforms in Apache HOP

Once you've configured your settings, you're ready to create your first workflows and transforms in Apache HOP. Here's a step-by-step guide to help you get started:

1. **Create a New Pipeline or Workflow**:
   - Click on the "plus" button located in the upper right corner of the interface.
   - From the dropdown menu, select either "Pipeline" or "Workflow".

2. **Start Working on the Canvas**:
   - Tap on the canvas area. A popup window will appear, allowing you to add actions.

3. **Explore Sample Projects**:
   - If you're not confident about starting from scratch, Apache HOP includes several samples to help you get started.
   - Switch your environment to "samples" and click on the "open file" button in the toolbar. This will give you access to a variety of pre-made samples.

4. **Locate Sample Files**:
   - All sample files are stored by default in the following directory: `\apache-hop-client-2.9.0\hop\config\projects`.
   - Navigate to this directory to find and open the sample projects, which can serve as a helpful reference or starting point for your own projects.


By following these steps, you'll be well on your way to creating effective and efficient workflows in Apache HOP. HOP offers a flexible and user-friendly environment to support your data processing needs.
