---
title: "Troubleshooting Version Control"
url: /refguide/troubleshoot-version-control-issues/
linktitle: "Troubleshoot Version Control"
weight: 20
description: "Presents a list of problems and fixes for version control issues."
tags: ["version control", "troubleshoot", "Studio Pro"]
aliases:
    - /howto/collaboration-requirements-management/troubleshoot-version-control-issues/
#If moving or renaming this doc file, implement a temporary redirect and let the respective team know they should update the URL in the product. See Mapping to Products for more details.
#The anchor #cannot-create-package below is mapped from Studio Pro, so it should not be removed or changed.
---

## 1 Introduction

Mendix Studio Pro contains a version control system that supports collaborating with team members. This document describes issues and limitations with Git version control and their workarounds.

For more information and general help on version control, see the following documents:

* [Version Control](/refguide/version-control/)
* [Using Version Control in Studio Pro](/refguide/using-version-control-in-studio-pro/)

## 2 Troubleshooting {#troubleshooting}

### 2.1 Getting an `The project contains changes that have not been committed yet. Please commit first before attempting to merge again. While you have not changes any files.` Error {#css-error}

{{% alert color="info" %}}

This issue is a known issue for Mendix 10.0 and above. For more information, see [10.0](/releasenotes/studio-pro/10.0/#css-ki) release notes.

{{% /alert %}}

You may get `The project contains changes that have not been committed yet. Please commit first before attempting to merge again. While you have not changes any files.` error when you feel there are no uncommitted changes. 

This happens due to line endings in CSS files not being handled properly. Thus, sometimes after reverting all changes or applying other version control operations the css.map files show up in the **Changes on Disk** dialog box.
To prevent the css.map be flagged as changed you can perform the following workaround:

1. Create `.gitattributes` file in **\theme-cache** folder of your application (if you have a Windows machine, make sure that a .txt extension is **not** applied by default). 
2. Add the following content to the file: 
   `* text eol=lf`
3. Save the file, and commit and push your changes.

From now, on Git will restore the files with the correct line endings and this will not be seen as a change.  

{{% alert color="info" %}}

The workaround corrects the issue for future commits. If you have other branches that were created before the fix, you need to add the same `.gitattributes` file in those branches as well.

{{% /alert %}}

### 2.2 Proxy Servers Are Not Supported

Studio Pro communicates to Git repositories by two means: LibGit2 library or Git command line interface (Git CLI). LibGit2 provides a nice and clean repository object model that is in intensive use during local repository operations. However, it is not performant enough when it comes to communication with remote Git servers. This is when Git CLI is used and Studio Pro switches to the client while performing fetch, pull and push operations (that is why the Git for Windows package, which ships Git CLI to your computer, is an integral part of Studio Pro installation). Therefore, any operation that requires transferring data to/from remote Git repositories, uses GitCLI client.

Unfortunately, Git for Windows is not synchronized with the system proxy settings by default, which means it may be tricky to integrate them into Studio Pro seamlessly. We are trying our best to provide integration as soon as possible, but as a workaround you can configure the proxy settings directly by either overriding **http_proxy**, **https_proxy** and **all_proxy** (for more information, see [Git documentation] (https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy)) or by setting the proxy URL in your local **.git/config** via the following commands:

* Configure the proxy settings:

  `git config --local http.proxy [protocol://][user[:password]@]proxyhost[:port]`

* Check that the setting has changed:

  `git config --local http.proxy`

You can also use `--global` modifier for applying the changes system-wise, but it is not recommended if you are using Git not only for Mendix development.

### 2.3 Getting an "Oops" Pop-Up Menu

You may get an "Oops" pop-up menu when setting a name and an email for Git in the **Preferences** dialog box or while committing. The problem might occur if there is no global Git config file on the PC yet. To fix this issue, you can use the Git command line client and do the following:

* Specify the user name:
  `git config --global user.name "<Name>"`
* Specify the email:
  `git config --global user.email "<Email>"`

Any of these commands will create a global Git config. Subsequent interactions via the Studio Pro interface will succeed from now on.

### 2.4 Merging Branches Results in No Changes Being Applied

When two branches are merged, changes sometimes do not come through.
To fix this issue, take the following steps:

1. Ensure `user.name` and `user.email` exist in your git config:
    * Specify the user name:
    `git config --global user.name "<Name>"`
    * Specify the email:
    `git config --global user.email "<Email>"` 
2. Ensure the git version on you machine is 2.43.x or above:
    * Check your installed git version by running the following command:
    `git version`.
    * If needed install the correct version from the [git website](https://git-scm.com/download/win).

### 2.5 Connection Problems When Cloning the Git Repository

If you face connection problems when cloning the Git repository using the **Open App** or **Download App** dialog box, the first thing to check is whether the URL of the remote Git repository is correct. It should not be copied from the browser address bar. Most Git services have a noticeable colored **Clone** button which provides the correct URL in a pop-up window. You should use this URL with Studio Pro.

### 2.6 Customer-Facing Issues

For Studio Pro developers to be able to troubleshoot issues that the customers face with beta Git support, Studio Pro provides a logging mechanism.

When filing a Git support issue with Mendix Support, attach the log files doing the following:

1. Navigate to the **Help** menu > **Open Log File Directory**:

   {{< figure src="/attachments/refguide/version-control/on-premises-git/troubleshoot-git-issues/open-log-file-directory-menu.png" alt="Download from Version Control Server dialog" >}}

2. Copy the file called *log.txt* into your ticket. You can also attach additional *log.X.txt* files if they exist.

### 2.7 Git Properties Useful for Troubleshooting

{{% alert color="warning" %}}
The properties described below might contain personal information. We advise you to make sure that all the private information is removed before sharing it. 
{{% /alert %}}

There are properties of the Git repository that provide you with the information useful for troubleshooting different issues. Execute the following in the command line in the app’s folder:

`git status -b` — provides information on the current state of the repository

`git remote -v` — lists the remotes specified for the repository

`git config --list --show-origin --show-scope` — provides information on user's Git config

### 2.8 Cannot Create Package from a Revision{#cannot-create-package}

Sometimes it is impossible to create a package from a certain revision. See below for a description of the most common case and troubleshooting steps for resolving it.

#### 2.8.1 Missing Metadata

##### 2.8.1.1 Issue

When you commit (and push) changes to the repository, Studio Pro adds an additional commit with so called metadata to a special refspec `.git/refs/notes/mx_metadata``. Making it a refspec means that you will not see this commit in your commits history.
This metadata contains the information needed to create a deployment package (for instance the version of Studio Pro that was used to create this revision).

To create a deployment package, Studio Pro downloads the specific revision into a temporary folder and then creates the package from there. Studio Pro checks the Mendix version of the selected revision to confirm that it is compatible with the version of Studio Pro. 

Without the metadata, Studio Pro cannot find out this information.

##### 2.8.1.2 Solution

You can force Studio Pro to recreate the metadata by doing the following:

1. Checkout the revision you want to create a deployment package from in a separate directory.
2. Open it with the correct version of Studio Pro.
3. Create a branch from it and switch to the new branch. 
4. Do a cosmetic change (for example, move an activity in a microflow a few pixels)
5. Commit and Push the changes from Studio Pro.

This will recreate the metadata and you should now be able to create a deployment package from this new revision.

### 2.9 Installed Git Version Does Not Meet Studio Pro's Requirement

Studio Pro requires a specific minimum version of Git (for more information, see [System Requirements](/refguide/system-requirements/)). If this version is not installed on your device, Studio Pro will show a message during a start-up indicating that certain features will not be available.

You can download and install a recent version of Git from [here](https://git-scm.com/downloads). 
