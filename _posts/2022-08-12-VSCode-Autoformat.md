---
keywords: software
description: "If you still manually format your code in your IDE, then this post is for you! Save your time by enabling your IDE to format the code for you — I will walk you through how to configure your VSCode to do auto-formatting on scripts in Python, Javascript and Rust."
title: "Delight Your VSCode Experience by Auto-formatting (Python, Javascript, Rust)"
toc: false
badges: true
comments: true
categories: [IDE, productivity]
image: images/2022-08-12-VSCode-Autoformat/cover.png
layout: post
---

## **Motivation**
---

When I was in a data science role, I didn’t give much attention to tools for code formatting. Ever since I transitted to a software engineer, I started appreciating the value of these tools. I find it even handy to automate them in an IDE.

The benefits are more than just saving time:

1. It saves you cognitive workload from manual code formatting, hence you can focus on things that are more important
2. It enforces uniform coding style with your teammates. Your team observes to the same coding style for free once they share the same configuration of a formatter.

As I personally use VSCode as my IDE, I will highlight how I install and configure formatter in my VSCode so that it is automatically triggered when saving a script in Python, Javascript (Typescript) or Rust.

It’s just a few simple steps, but your experience will never be the same! Now I cannot part without auto-formatting in my daily coding routine.

{% include note.html content='You may wonder the differences between Linter and Formatter.
Linter flags bugs and violation of best practices, while Formatter enforces a coding style on your code. Best practices could be related to code style, so formatter could help address a small part of best practices. I may open a separate blogpost about Linters in the future.' %}

## **Python**
---

1. Install [Black Formatter](https://marketplace.visualstudio.com/items?itemName=ms-python.black-formatter) in Extension Marketplace (shortcut is <span style="color:grey">Command + Shift + X </span> in Macbook)
    
    ![Screenshot 2022-07-30 at 3.14.15 PM.png]({{ site.baseurl }}/images/2022-08-12-VSCode-Autoformat/black_formatter.png)
    
2. Open a project in VSCode, navigate to <span style="color:orange">“User Settings”</span> (shortcut is <span style="color:grey">Command + ,</span> in Macbook), then <span style="color:orange">“Workspace”</span> tab, then <span style="color:orange">“Open Settings (JSON)”</span> button, as shown in the screenshot. Now you are in the editing page of `settings.json`, which configures VSCode’s functionalities specific to your project.
    
    ![Screenshot 2022-07-30 at 3.21.16 PM.png]({{ site.baseurl }}/images/2022-08-12-VSCode-Autoformat/workspace_settings.png)
    
3. Paste the following code snippet in `settings.json`:
    
    ```json
    {
        "[python]": {
            "editor.defaultFormatter": "ms-python.black-formatter",
            "editor.formatOnSave": true
        }
    }
    ```
    
4. Optional: Make a new file named `pyproject.toml` in your project folder. It customises the formatting rules for Black Formatter. For example, if I want the number of characters per line to be no more than 80, I can paste the following snippet in the file:
    
    ```json
    [tool.black]
    line-length = 80
    ```

Voila! Auto-formatting is ready to run for Python script! 🔥

Here is a short demo of how it behaves when I save a Python file:

![Screen Recording 2022-07-30 at 3.07.44 PM.mov]({{ site.baseurl }}/images/2022-08-12-VSCode-Autoformat/black_demo.gif)

<br>

{% include note.html content='Besides Black Formatter, there are other Python formatters available in Extension Market. For example, Isort for importing sorting.
Alternatively, these formatters comes as a package, so that you can apply them in command line. For example, Black Formatter is making use of Black package underneath the hood. Flake8 is another package for code formatting but it doesn’t have an extension in VSCode. Feel free to explore them on your own!' %}

## **Javascript (or Typescript)**
---

1. Install [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) in Extension Marketplace
    
    ![Screenshot 2022-07-30 at 3.40.03 PM.png]({{ site.baseurl }}/images/2022-08-12-VSCode-Autoformat/prettier_formatter.png)
    
2. Navigate to the editing page of `settings.json` and paste the following code snippet:
    
    ```json
    {
        "[javascript]": {
            "editor.defaultFormatter": "esbenp.prettier-vscode",
            "editor.formatOnSave": true
        },
        "[typescript]": {
            "editor.defaultFormatter": "esbenp.prettier-vscode",
            "editor.formatOnSave": true
        }
    }
    ```
    
3. Optional: Make a new file named `prettierrc.json` in your project folder. It customises the formatting rules for Prettier. For example, if I want to enforce rules like single quote in string and double spaces in tab, I can paste the following snippet into the file:
       
    ```json
    {
        "singleQuote": true,
        "trailingComma": "all",
        "printWidth": 100,
        "tabWidth": 2,
        "useTabs": false
    }
    ```
    

Now auto-formatting is ready for Javacsript! Here is a demo:
    
![Screen Recording 2022-07-30 at 3.51.57 PM.mov]({{ site.baseurl }}/images/2022-08-12-VSCode-Autoformat/prettier_demo.gif)


## **Rust**
---

1. Install [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer) in Extension Marketplace (Rust analyzer is not just a formatter. It supports many features such as error popup, datatype detection)
    
    ![Screenshot 2022-07-30 at 3.57.36 PM.png]({{ site.baseurl }}/images/2022-08-12-VSCode-Autoformat/rust_analyzer.png)
    
2. Navigate to the editing page of `settings.json` and paste the following code snippet:
    
    ```json
    {
        "[rust]": {
            "editor.defaultFormatter": "rust-lang.rust-analyzer",
            "editor.formatOnSave": true
        }
    }
    ```
    
1. Optional: Customising rules in rust-analyzer requires more effort, you can refer to [this documentation](https://github.com/rust-lang/rust-analyzer/blob/master/docs/user/manual.adoc#configuration) for details. Its default setting suffices in most use cases. 


Here is the final demo to showcase the auto-formatting in Rust script:
    
![Screen Recording 2022-07-30 at 4.07.54 PM.mov]({{ site.baseurl }}/images/2022-08-12-VSCode-Autoformat/rust_analyzer_demo.gif)
    

## **Conclusion**
---

In this post, I have shown you:

1. how to install different formatters in VSCode Extension Marketplace
2. how to customise your formatters with its configuration file
3. how to configure the formatter to be applied upon file saving in VSCode

You can further explore the functionalities of each formatters and its integration with VSCode.

As an advanced setup, you can even apply multiple formatters on a script upong saving (e.g. applying Black, Flake8 and Isort formatters all in once). If there are enough of readers interested in this trick, I could write an additional blogpost for that!

## **References**
---

1. [How To Enable Linting on Save with Visual Studio Code and ESLint, DigitalOcean](https://www.digitalocean.com/community/tutorials/workflow-auto-eslinting)
2. [Linting and Formatting with ESLint in VS Code](https://morioh.com/p/7a567e1c0e1b)
3. [Styling and Formatting Code - Made With ML](https://madewithml.com/courses/mlops/styling/)
4. [How to run cargo fmt on save in vscode?](https://stackoverflow.com/questions/67859926/how-to-run-cargo-fmt-on-save-in-vscode)
5. [Support multiple formatters for a single file · Issue #142904 · microsoft/vscode](https://github.com/microsoft/vscode/issues/142904)
6. [Automatically Execute Bash Commands on Save in VS Code](https://betterprogramming.pub/automatically-execute-bash-commands-on-save-in-vs-code-7a3100449f63)