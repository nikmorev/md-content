# Package registry in GL

## How to set up one package registry for multiple packages

Let's say there's a gitlab account with on domain **gl.company.com** and there's a group of repos called **studyshore**, all repos in studyshore are private, and there's a repository **fe-utils** that consists of multiple utilities that are used in few different places and can be installed via `npm i`.  
Gitlab proposes it's own free package registry for different packages, including npm packages, and here are the steps for creating package registry on gitlab, publishing npm packages to this registry, and installing packages from such registry.

### Creating package registry

1. Create an empty project (for example **frontent-packages**);
2. Generate access token (Go to frontent-packages -> settings -> access tokens) and save it somewhere safe (let's imagine that our token is **d32cff2**). We gonna need this tokken to publish packages via CLI and to install packages via `npm i` (in case the package registry is private).  

3. Find the project ID of repository used as package registry (in our example it's **frontend-packages** registry). ID is located on main page of **frontend-packages** under the name of the project (on top of the page).

So for publishing the package we gonna need only access token and ID of project used as package registry (**frontend-packages**) in our example.

### Publishing package via CLI

To publish some package via CLI, the one should follow the next steps:

1. Create **.npmrc** file in package folder (for example our new package is called **utils**) and add there few lines:

```
@scope:registry=https://your_domain_name/api/v4/projects/your_project_id/packages/npm/
//your_domain_name/api/v4/projects/your_project_id/packages/npm/:_authToken="${NPM_TOKEN}"
```
For example our gitlab domain is **gl.company.com** and project (package registry, **frontend-packages** in our example, with ID = 205) is in group **studyshore**, the **.npmrc** will look like:
```
@studyshore:registry=https://gl.company.com/api/v4/projects/205/packages/npm/
//gl.company.com/api/v4/projects/205/packages/npm/:_authToken="${NPM_TOKEN}"
```
2. In **package.json** of **utils** add correct name of the project `"name": "@studyshore/utils"` and publishConfig like:
```
"publishConfig": {
    "@studyshore:registry": "https://gl.company.com/api/v4/projects/205/packages/npm/"
}
```  

3. Run `NPM_TOKEN=your_token npm publish` (`NPM_TOKEN=d32cff2 npm publish`) to publish **utils** package to package registry (**frontend-packages**). your_token - it's an access token (for **frontend-packages** project)


### Publishing package via CI/CD

To publish some package via CI/CD of gitlab, the one should follow the next steps:

1. Create **.npmrc** file in package folder (for example our new package is called **utils**) and add there few lines:

```
@scope:registry=https://your_domain_name/api/v4/projects/your_project_id/packages/npm/
//your_domain_name/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}
```
For example our gitlab domain is **gl.company.com** and project (package registry, **frontend-packages** in our example, with ID = 205) is in group **studyshore**, the **.npmrc** will look like:
```
@scope:registry=https://gl.company.com/api/v4/projects/205/packages/npm/
//gl.company.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}
```  
The \${CI_PROJECT_ID} and \${CI_JOB_TOKEN} are predefined variables that are available in the pipeline and do not need to be replaced.


2. In the GitLab project that houses your **.npmrc** and **package.json**, edit or create a **.gitlab-ci.yml** file. For example:
```
image: node:latest

stages:
  - deploy

deploy:
  stage: deploy
  script:
    - echo "//${CI_SERVER_HOST}/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}">.npmrc
    - npm publish
```

Package should now publish to the Package Registry when the pipeline runs.


### Install a package

To install a package from gitlab package registry, follow the next steps:  

1. Authenticate to the Package Registry if package registry (**frontend-packages** in our example) is private. Otherwise, skip this step.

```
npm config set -- //your_domain_name/api/v4/packages/npm/:_authToken=your_token

npm config set -- //your_domain_name/api/v4/projects/<project-id>/packages/npm/:_authToken=your_token
```  

For our example it's gonna look like:
```
npm config set -- //gl.company.com/api/v4/packages/npm/:_authToken=d32cff2

npm config set -- //gl.company.com/api/v4/projects/205/packages/npm/:_authToken=d32cff2
```

**OR** add to .npmrc
```
@scope:registry https://your_domain_name/api/v4/packages/npm/
//your_domain_name/api/v4/packages/npm/:_authToken=${NPM_TOKEN}
//your_domain_name/api/v4/projects/<project-id>/packages/npm/:_authToken=${NPM_TOKEN}
```
For our example it's gonna look like:
```
@studyshore:registry https://gl.company.com/api/v4/packages/npm/
//gl.company.com/api/v4/packages/npm/:_authToken=${NPM_TOKEN}
//gl.company.com/api/v4/projects/251/packages/npm/:_authToken=${NPM_TOKEN}
```
and run in terminal: `NPM_TOKEN=token npm i package-name`  
For our example it's gonna look like: ``NPM_TOKEN=d32cff2 npm i package-name``

2. Set the registry
```
npm config set @scope:registry https://your_domain_name.com/api/v4/packages/npm/
```
For our example it's gonna look like:
```
npm config set @studyshore:registry https://gl.company.com/api/v4/packages/npm/
```

3. Install the package
```
npm install @scope/my-package
```
For our example it's gonna look like:
```
npm install @studyshore/utils
```




## References
* [https://docs.gitlab.com/ee/user/packages/npm_registry/index.html](https://docs.gitlab.com/ee/user/packages/npm_registry/index.html)
* [https://www.youtube.com/watch?v=ui2nNBwN35c](https://www.youtube.com/watch?v=ui2nNBwN35c)
