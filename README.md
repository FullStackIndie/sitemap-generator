# sitemap-generator

A C# console application CLI Tool and GitHub Action that generates an XML site map for a website. Sitemaps are important because they help Search engines index and navigate your website and rank you higher in SEO.

## Description

This is a simple site map generator that takes a URL as input and creates a site map of all the pages on the website.

This is meant to be used in your GitHub Actions to generate a current site map for your website as part of your deployment workflow. You can also download it and run locally on your computer.

This app/action will search your domain for anchor tags that have **"href"** or **"data-href"** attributes and add them to the sitemap. **"data-href"** is used in the event you have modals or any dynamic javascript that updates the **"href"** attribute dynamically after the page is loaded. This will ensure that all links within your domain are included in the sitemap.

***

## Use In GitHub Actions

I would recommend using this action in your deployment workflow after you have deployed to production. This will ensure that your sitemap is always up to date with the latest changes to your website.

- url: the url you want to generate a site map for. This is required.

- cache: - Should the sitemap be uploaded as an artifact to be used in another job. The default value is false. If you don't upload the artifact you can acess the sitemap path (where it was created) using this variable format **${{ steps.step-id.outputs.sitemap }}**

Example - **${{ steps.sitemap.outputs.sitemap }}**
```
    steps:
      - name: Create a Sitemap
        id: sitemap
        uses: FullStackIndie/sitemap-generator@v1.4.0
        with:
          url: ${{ inputs.url }}
          cache: ${{ inputs.cache }}
          cache-key: ${{ inputs.cache-key }}
```
In this example github action the step id is "sitemap" so to access the sitemap path you would use this variable **${{ steps.sitemap.outputs.sitemap }}** [ Note: the output variable is also called sitemap ]


- cache-key: - the cache key to upload the site map to. The defualt value is 'sitemap'. You will use the same key to download the sitemap in your deployment workflow.

1. The sitemap will be named **sitemap.xml**
2. Logs are named **sitemap_generator_logs.txt**

### To create and upload a sitemap as an artifact to be downloaded in another job

```
name: Create a Sitemap

on:
  push:
    branches: [main]

  pull_request:
    branches: [main]

jobs:
  create-site-map:
    runs-on: ubuntu-latest
    steps:
      - name: Create a Sitemap
        uses: FullStackIndie/sitemap-generator@v1.4.0
        with:
          url: https://portfolio.fullstackindie.net
          cache: 'true'
          cache-key: sitemap
```

### To download the sitemap use this action in your workflow

```
name: Create a Sitemap

on:
  push:
    branches: [main]

  pull_request:
    branches: [main]

jobs:
  download-site-map:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v3.5.2

    - name: Download Sitemap
      uses: actions/download-artifact@v3
        with:
          name: sitemap
          path: ./ 
```

### If using Asp.Net Core and want to save sitemap to wwwroot so you can access it at https://portfolio.fullstackindie.net/sitemap.xml your paths may look like this

```
- name: Download Sitemap
  uses: actions/download-artifact@v3
    with:
    name: sitemap
    path: ./MyProject/MyProject/wwwroot/    # normal Asp.NetCore Folder Structure
    path: ./MyProject/wwwroot/               #  Asp.NetCore Single Folder Structure
    path: ./app/wwwroot/             #  Asp.NetCore Docker Folder Structure
```

### Use site map in the same job with other actions. Use the output of *sitemap-id* to access the path of where the sitemap is located. ${{ steps.sitemap-id.outputs.sitemap }}

- sitemap - is the variable that contains the path to the sitemap. You can use this variable to upload the sitemap to AWS S3 or any other storage service, or deploy to your server.

```
name: >-
  Create SiteMap for Website and upload sitemap to S3

on:
  push:
    branches: [main]

  pull_request:
    branches: [main]

jobs:
  create-sitemap-and-upload:
    runs-on: ubuntu-latest
    steps:
      - name: Create a Sitemap
        id: sitemap-id
        uses: FullStackIndie/sitemap-generator@v1.4.0
        with:
          url: https://portfolio.fullstackindie.net
          cache: "false"
          cache-key: sitemap

      - name: Sync files to Aws S3
        run: |
          aws s3 sync ${{ steps.sitemap-id.outputs.sitemap } 
          s3://my-bucket/

```

***

## Install Locally on Windows and Linux [[Requires .Net 7 SDK or the .Net 7 Runtime]](<https://dotnet.microsoft.com/en-us/download/dotnet/7.0>)

- Comes with dockerfile for easy deployment for Windows, Linux, MacOS or the Cloud. I havent tested using it with Docker yet so no documentation yet
- **Make sure you have permissions to create files and folders in the directory you specify when using the -p or --path flag to save sitemap.xml.
    I got errors on Windows after I changed the Console implemenatation. Will debug the reason eventually hopefully and fix it**

***

### Linux / Ubuntu

Clone Repo into an empty Directory such as `/opt/sitemap-generator/` or `~/Workspace/Tools/sitemap-generator/`

```
mkdir /opt/sitemap-generator
cd /opt/sitemap-generator
git clone https://github.com/FullStackIndie/sitemap-generator.git .
dotnet publish ./SiteMapGenerator.csproj -c Release -r linux-x64 -o ./build
cd ./build && mv SiteMapGenerator sitemap
```

To add sitemap-generator to the path in Linux, you can use one of the following methods:

1. If you want sudo/root permissions 

Make a symlink in /usr/bin (or /usr/local/bin) directory: 

```
sudo ln -s /opt/sitemap-generator/build/sitemap /usr/bin/sitemap
export $PATH=$PATH:/usr/bin/sitemap
```

You have to run sitemap with sudo

***

2. If you don't need sudo/root permission:

```
mkdir -p $HOME/.local/bin
ln -s /opt/sitemap-generator/build/sitemap $HOME/.local/bin/sitemap
export PATH=$PATH:$HOME/.local/bin
```

Restart a new shell and You should be able to type `sitemap` and see the help menu. If so installation is successful.

If path isnt persisted in new shell check out the docs for [Ubuntu](https://help.ubuntu.com/community/EnvironmentVariables#Persistent_environment_variables)
***

### Windows GitBash

```
cd /c/ && mkdir sitemap-generator
cd /c/sitemap-generator && git clone https://github.com/FullStackIndie/sitemap-generator.git .
dotnet publish ./SiteMapGenerator.csproj -c Release -r win-x64 --self-contained true -o ./build
cd ./build && mv SiteMapGenerator.exe sitemap.exe
```
### Windows CMD

```
cd C:\ && mkdir sitemap-generator
cd C:\sitemap-generator && git clone https://github.com/FullStackIndie/sitemap-generator.git .
dotnet publish ./SiteMapGenerator.csproj -c Release -r win-x64 --self-contained true -o ./build
cd ./build && rename SiteMapGenerator.exe sitemap.exe
```

To add sitemap-generator to the path in Windows, you can use the following method:

Open the Start Search, type in “env”, and choose “Edit the system environment variables”

Click the “Environment Variables…” button.

Under the “System Variables” section (the lower half), find the row with “Path” in the first column, and click edit.

The “Edit environment variable” UI will appear. Here, you can click “New” and type in the new path you want to add.

```
C:\sitemap-generator\build
```

Click OK on all windows.

Open CMD prompt and type `echo %PATH%`

You should be able to type `sitemap` and see the help menu. If so installation is successful.

***

## CLI Usage

### Example

```
Usage: sitemap <url> [options] -p -f -L 
sitemap https://www.example.com -p "/directory/to/save/sitemap" -f Daily -L Information
```

- First argument is the URL of the website you want to generate a site map for. The URL must be a valid URL and must include the protocol such as `https://` or `http://`.
 **[ Although the url is required you do not need to explicitly use it in the CLI ]**
- -p is the path to save the sitemap.xml file. This is optional and if not specified the sitemap.xml will be saved in the current directory.
- -f is to specify the frequency of how often your website changes. Default is Daily. As of right now you can only specify 1 value and all 
    links in the sitemap will be updated with that value
- -L is the log level you want to logged to the Console and the log file that is generated. SiteMap Generator can only save log file to the current directory as of now.

| Options | Required | Default | Example Value
| :-------------- | :-------------: | ------------: | -----------: |
| url            | true     | none       | `https://www.example.com` or `http://www.example.com`
| -p or --path     | false         | Current Directory      | **'.'** or **'/var/www/html/blog'** or **'C:\Users\Me\Documents\My Website'**
| -L or --logLevel     | false          | Information       | Verbose, Debug, Information, Warning, Error
| -f or --frequency    | false          | Daily       | Always, Hourly, Daily, Weekly

### Linux Example

```
# run globally if added the Path
cd /var/www/html
sudo sitemap url https://www.example.com -p /var/www/html -f Daily -L Debug
or
sudo sitemap https://www.example.com -p /var/www/html -f Daily -L Debug

cd /var/www/html
sitemap url https://www.example.com -p /var/www/html -f Daily -L Debug
or
sitemap https://www.example.com -p /var/www/html -f Daily -L Debug

# run from direcory where installed
./sitemap url https://www.example.com -p /var/www/html -f Daily -L Debug
or
./sitemap https://www.example.com -p /var/www/html -f Daily -L Debug
```

### Windows GitBash Example

```
# run globally if added the Path
cd ~/Documents/My Website
sitemap url https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug
or
sitemap https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug

# run from direcory where installed
./sitemap.exe url https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug
or
./sitemap.exe https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug
```

### Windows CMD Example

```
# run globally if added the Path
cd ~/Documents/My Website
sitemap url https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug
or
sitemap https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug

# run from direcory where installed
./sitemap.exe url https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug
or
./sitemap.exe https://www.example.com -p /c/Users/Me/Documents/My Website -f Daily -L Debug
```

Restart CMD, PowerShell, or GitBash as Administrator if having 'Access Denied' issues when saving sitemap.xml
