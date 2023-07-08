## SIMULATING CICD WITH AWS CODE COMMIT, CODE BUILD & CODE PIPELINE (for a node.js application)


## PREREQUISITES:
- AWS account.
- Install Git on your local machine.


## STEP 1: SET UP AWS CODECOMMIT:
- Go to the AWS Management Console and search for "CodeCommit".
- Create a new repository in CodeCommit
- navigate to IAM user and secure the credentials (Username and password) to access AWS code commit.


## STEP 2: SET UP GIT AND PUSH TO CODECOMMIT:

* Start by initializing a Git repository on your local machine. Open a terminal, navigate to your project directory, and run the command `git init` to set up the repository.

* Add your Node.js app files to the Git repository. Use the command `git add .` to include all files in the repository.
  After populating the node.js files with the required contents, npm and node.js was installed. `npm init ` was run to initiailise the project directory and create the package.json file. `npm install express ` was run afterwards.

* Commit your changes to the repository. Execute the command `git commit -m "Initial commit"` to create a commit with an appropriate commit message.

* Next, establish a connection to the remote CodeCommit repository. Obtain the "HTTPS clone URL" from the CodeCommit repository page. Use the command `git remote add origin <clone_url> ` to link your local repository to the remote repository, replacing <clone_url> with the actual URL.

* Finally, push your code to CodeCommit. Execute `git push origin master` to push the committed changes to the master branch. If prompted, provide your AWS CodeCommit credentials to complete the push.

By following these steps, I successfully configured Git, added my Node.js app files to the repository, commited my changes, set up the remote repository connection, and pushed my code to CodeCommit.

![TRIO](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/c005b46e-4032-45b6-9488-5b9e5c36d875)

## STEP 3: SET UP AWS CODEPIPELINE:

* Open the AWS Management Console and search for "CodeBuild".

* Click on "Create build project" to start the project creation process.

* Configure the source settings:
  Select the source provider, AWS CodeCommit in this case
  Choose the repository ("Trio") and branch (master) to use.
  Specify the buildspec file location (buildspec.yml)

* Configure the environment settings:
  Select the operating system, runtime, and compute type for the build environment.
  Choose whether to use a managed image or a custom Docker image. Managed image was used.
  Specify any additional environment variables needed for your build process.
  
![env variable](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/92e08a6c-d247-451d-9b28-78979594e1fc)

* Configure the optional notifications:
  Specify whether to receive build notifications via Amazon SNS.
  Choose the SNS topic and events for which you want to be notified.

* Review the project settings to ensure they match your requirements.

* Click "Create build project" to create the AWS CodeBuild project.

## STEP 4: INCLUDING THE CODE BUILD ABOVE IN AWS CODEPIPELINE

* Click on the "Create pipeline" button and proceed with the following steps:
  Provide a name for your pipeline.
  Choose "AWS CodeCommit" as the source provider.
  Select the desired repository and branch to use.
  Opt for "AWS CodeBuild" as the build provider. Select the earlier created project (node-js-app)

![build-projects](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/5a1d3d8f-b902-4ad2-8316-7e16621c6db7)

* The deployment stage was skipped. We shall get to it.
 
* Review the settings of the pipeline and click "Create pipeline" to create it.
  
![Pipeline](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/603dde70-3637-453d-9809-abd3acf42b0d)

* Upon a new commit to the AWS code commit repository created, the pipeline is triggered.

* The pipeline stages was successful
  
![result 2](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/8c92a5ed-a107-46dc-b7f6-1cadbe966ac3)

![result 3](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/4364680f-8bbf-40cc-afc4-255fb3fd5883)

![result 4](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/807f6a08-cdd9-4d37-8372-03cb2e0ddd91)

![result 5](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/9983f2a2-241b-4801-9a0a-2c68071c67e8)

![result 6](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/7f303c45-a3e6-48e6-8330-13d1cc018de2)

![result 7](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/c6e41178-0f3f-4817-af89-27a359f3c71c)

![result 8](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/3d2eeba5-798c-4e9e-93e5-52e318daa8db)


* The image was successfully pushed to my private repo in ECR.
* 
![image in ecr](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/4ca4148d-298b-4fe6-92a6-e046bb9d601e)

################################################################################################

**THINGS TO NOTE FOR REFERENCE SAKE**

The project was executed in two stages;
- Running the node.js application 
- Building a docker image for the nodejs application from Dockerfile and pushing the image to AMAZON ECR.
 
Here's a view of the build stages as specified in the buildspec.yml file

![spec 1](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/8c25b126-7b1b-4234-a50c-29de7f0d355e)
![spec 2](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/ca3963a0-25d7-436d-b2e2-169bd61f927c)

## CHALLENGES;

1.
While running the node.js application, the pipeline got stuck after running the "npm run start" stage.

![stuck pipeline](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/8435f3af-93a3-4a52-8bc7-cf2d5db78d12)

I discovered that the npm scripts specified ` node index.js ` as the command to run in the start stage. This hijacks the session as the command is run in the foreground thus not returning control back to the pipeline. As a result, the subsequent stages in your pipeline are not executed.

![node index](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/bbe83c62-e847-43f1-b9a6-39e86a88e5e6)

## SOLUTION.

- I utilised "pm2", a process manager to start the node index.js command in the background. ` pm2 start index.js` instead of ` node index.js `
  
![packagejson](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/f00d58d1-a82e-4b4d-9fe1-f6abe451638d)

- The build stage of the buildspec.yml file was also updated to install pm2 `- npm install pm2 --save-dev `

2.
An error was encountered when I attempted to push the image to Amazon ECR as I created a public repository earlier. I had to resort to using a private repository in ECR and modified my code snippet login command. The error is as shown below. Failed to get authorization tokens even after the necessary policies (AWSEC2ContainerRegistryFullAccess) were attached to the code build service role.

![public error](https://github.com/Babbexx-22/AWS-CICD-Pipeline-Project/assets/114196715/fff87c79-34a0-46e2-ac01-2c9ba856420a)

**END**
