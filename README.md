# DevOps Infrastructure Automation : Continuous Development with Jenkins, GitHub and Docker
## Exercises for "DevOps Engineering Warriors"

## Exercise I : Jenkins Freestyle Project
Go to your Jenkins Server : https://<YOUR_JENKINS_SERVER>/

Create a new folder with the following syntax <Firstname_NameOfYourFavoriteFruit_Codeuse> by clicking on "New Item" button

<img src="img\newItem.PNG">

then fill out all the required fields to create your folder as shown below :

<img src="img\folderCreation.PNG">

Click on "OK" button and a new Configuration page appears on UI, just click on "Save" button

Inside your Jenkins folder, create your freestyle project

<img src="img\freestyleCreation.PNG">

Click on "OK" button and a new Configuration page appears on UI,

Now discard old builds to avoid consuming a big amount of disk space on the Jenkins master as follows

<img src="img\discardOldBuilds.PNG">

Add a GitHub project url related to your jenkins job purposes
- https://github.com/rovland/descodeuses_part2

<img src="img\addGitHubProject.PNG">

Add a string parameter to your project

<img src="img\addStringParams.PNG">

Set NAME as a string parameter in your project

<img src="img\nameStringParam.PNG">

Set COUNTRY as a choice parameter

<img src="img\countryChoiceParam.PNG">

Set CAPTCHA as a Boolean parameter

<img src="img\humanCheck.PNG">

Select "Build Environment" tab and click on "Execute Shell"

<img src="img\executeShell.PNG">

Add the following shell code

<img src="img\shellCodeToExecute.PNG">

Click "Save" button before leaving this configuration page and then click on "Build with Parameters" on the left side

<img src="img\buildWithParameters.PNG">

The following UI appears on the screen and have a fun to play with it

<img src="img\macronExample.PNG">

After clicking on "Build", check out your jenkins job log

- Step#1

<img src="img\macronBuildSelected.PNG">

- Step#2

<img src="img\macronConsoleOutputClick.PNG">

- Step#3

<img src="img\macronConsoleOutputLog.PNG">

**Questions :** what will happen in your build history if you execute 5 times this jenkins job ? do you know the root cause of this behavior ?

Now go back to your job configuration page and add a new shell to play with Jenkins env variables as follow

<img src="img\jenkinsPlayWithEnvVars.PNG">

Now save your modifications and execute a new build then have a look at the console output

Humm it's weird!!! .... **JENKINS BUILD_UR** variable seems not working ...Fix this issue by checking out the link : https://<YOUR_JENKINS_SERVER>/env-vars.html/

Also add in your job configuration the following code :

``` shell
echo "JENKINS BUILD_TAG = $BUILD_TAG"
```

Now, separate above two shell codes in two jenkins jobs (example : job#1 and job#2) and find a way to automatically execute job#2 when job#1 is finished

- Tips : pay attention to setting listed into "Post Build Actions" in your jenkins job configuration

<img src="img\postBuildActions.PNG">

To finish, find a way to automatically schedule the execution of your jenkins job#1 created above

- Tips : have a look at "Build Periodically with parameters" option into your job configuration and also be familiar with cron syntax : https://en.wikipedia.org/wiki/Cron

<img src="img\triggerWithParams.PNG">

Example : I would like to automatically run a jenkins job (job#1) each day at 2H30 PM with the following parameters NAME=Bernard Arnaud and COUNTRY=FRANCE

``` shell
TZ=Europe/Berlin
30 14 * * 1-5 %NAME=Bernard Arnaud;COUNTRY=FRANCE
```

Save your changes and check out your jenkins job "Build History"

Now update your cron syntax to add CAPTCHA as parameter


## Exercise II : Jenkins Pipeline (Groovy Script)

Sign-in to your GitHub account... If you don't have a GitHub account, please create it
- Tips : GitHub website => https://github.com

If you're creating your GitHub repo for the first time, please do not forget to verify your GitHub email address by looking at your mail messages

Send your GitHub user name to rovland in order to grant you access to collaborate to the following repo => https://github.com/rovland/descodeuses_part2

<img src="img\user_name_github_example.JPG">

Once rovland added you, accept the invitation here : https://github.com/rovland/descodeuses_part2/invitations

<img src="img\invitation_format_onGitHub.JPG">

Install GitHub Desktop : https://desktop.github.com/

Install Visual Studio Code : https://code.visualstudio.com/download

Clone this project on local using Github Desktop: https://github.com/rovland/descodeuses_part2

Open "descodeuses_part2" project in Visual Studio Code and create a folder with the syntax : [FirstName]\_[NameOfYourFavoriteFruit]\_Codeuse

<img src="img\folderCreationJenkinsPipeline.PNG">

Under your project, create a file called : Jenkinsfile

<img src="img\newJenkinsFileCreated.PNG">

Edit your Jenkinsfile as mentioned below and in the code, please replace my email address to your own :

```groovy
properties(
    [[
        $class: 'BuildDiscarderProperty', 
        strategy: [$class: 'LogRotator', 
        artifactDaysToKeepStr: '', 
        artifactNumToKeepStr: '3', 
        daysToKeepStr: '', 
        numToKeepStr: '3'
    ]],

    disableConcurrentBuilds(),
    
    parameters([
        string(
            defaultValue: '', 
            description: "Enter your Firstname LastName", 
            name: 'NAME'
        ),
        choice(
            choices: ['FRANCE', 'GERMANY', 'SPAIN', 'SENEGAL', 'ITALY'],
            description: """
            Select your country
            Example : France, Germany, Spain, Senegal, Italy
            """, 
            name: 'COUNTRY'
        ),
        booleanParam(
            defaultValue: false, 
            description: "Are you a human ?", 
            name: 'CAPTCHA'
        )
    ])
])

node("master") {
    
    checkout scm
    
    def cmd
    def cmdResult
    def errMessage


try {

    stage("Know Your Customer"){
        
        if (currentBuild.result != 'FAILURE') {
        
            sh "
                #!/bin/bash

                echo "********* START: KNOW YOUR CUSTOMER *********"

                [ ! -z "$NAME" ] && echo "My name is $NAME and I live in $COUNTRY" || (echo "Please enter your name"; exit 1)

                [ $CAPTCHA == true ] && echo "Youpii I'm a human" || echo "What a hell was that, it's a robot"

                echo "********* END : KNOW YOUR CUSTOMER *********"
            "
        } else {

            println "The stage 'Know Your Customer' was skipped"
        }
    }

    stage("Getting Jenkins Env Vars"){
        
        if (currentBuild.result != 'FAILURE') {
        
            sh '
                #!/bin/bash

                echo "****** START : JENKINS ENVIRONMENT VARIABLES ******"

                echo "JENKINS_HOME =" ${env.JENKINS_HOME}
                echo "JENKINS BUILD_URL = $BUILD_URL"
                echo "JENKINS_BUILD_NUMBER = $BUILD_NUMBER"
                echo "JENKINS BUILD_TAG = $BUILD_TAG"
                echo "JENKINS NODE_NAME = $NODE_NAME"

                echo "****** END : JENKINS ENVIRONMENT VARIABLES ******"
            '
        } else {

            println "The stage 'Getting Jenkins Env Vars' was skipped"
        }
    }

} catch(Exception err){

        currentBuild.result = 'FAILURE'
        errMessage = err.getMessage()
        
        throw err
    
    } finally {

        if ( currentBuild.result == 'FAILURE' ) {

            emailext attachLog: false,
            body: """Dear colleague, <br><br> 
                    Unfortunately we found an issue while executing your first jenkins pipeline job <br>
                    <i style="color:red" >$errMessage</i><br>
                    Please check-out the following link for more information : ${env.BUILD_URL} <br><br>
                    Run Live, Run Simple <br> 
                    DevOps Infrastructure Team
                    """,
            //recipientProviders: [buildUser()],
            to:"<ENTER_YOUR_EMAIL>",
            subject: "${env.JOB_NAME} => Status: ${currentBuild.currentResult}"

        } else {
                
            currentBuild.result = 'SUCCESS'

            emailext attachLog: false,
            body: """Dear colleague, <br><br> 
                    I'm excited to inform you that your first jenkins pipeline job was successfully executed <br><br>
                    Please check-out the following link for more information : ${env.BUILD_URL} <br><br>
                    Run Live, Run Simple <br> 
                    DevOps Infrastructure Team
                    """,
            //recipientProviders: [buildUser()],
            to:"<ENTER_YOUR_EMAIL>",
            subject: "${env.JOB_NAME} => Status: ${currentBuild.currentResult}"
            
        }
    }
}
```
Save your Jenkinsfile and push it (including your folder as well) into master branch

Now on Jenkins website, create your new jenkins pipeline job under your jenkins folder

<img src="img\createJenkinsPipeline.PNG">

Click "Ok" button

On the configuration page, go to "Pipeline" tab and configure your project as shown below :

<img src="img\jenkinsfileConfigurationPart1.PNG">

Scroll down at the bottom and change the value of 'Script Path' field and set the full path of your Jenkinsfile submitted to GitHub repository

<img src="img\scriptPathJenkinsfile.PNG">

Save your changes and build the job

Find a way to fix all errors occur by editing your Jenkinsfile on local

Now if you've fixed all errors occur , submit your modified Jenkinsfile into master branch and re-test your job on jenkins

Make sure that you've configured and saved your github personal token : https://github.com/settings/tokens  (Make sure to copy your personal access token now. You won’t be able to see it again!)

Add your GitHub personal token into Jenkins Credentials : https://<YOUR_JENKINS_SERVER>/credentials/store/system/domain/_/newCredentials

Update your Jenkinsfile with the groovy code below, push it into master branch and fix all error occur

- Tips : Replace the credentialsId in the code below with your own

- Tips : You can either validate your jenkinsfile by clicking on 'Replay' button in your build history on Jenkins or you can install a jenkins plugin into VS Code to validate your file before committing (https://www.jenkins.io/blog/2018/11/07/Validate-Jenkinsfile/ or https://dev.to/nicoavila/how-to-validate-your-jenkinsfile-locally-before-committing-334l )

```groovy

    withCredentials([
        usernamePassword(
            credentialsId: '6558aa6er-9608o-4d97v-b0a3l-aa7313e9da68and', 
            passwordVariable: 'tech_pwd', 
            usernameVariable: 'tech_user'
        )
    ]) {

        stage("Reading Tech User"){
            
            if (currentBuild.result != 'FAILURE') {
            
                cmd = """
                    #!/bin/bash

                    echo "****** START : READING TECH USER ******"

                    echo "TECHNICAL_USER =" ${tech_user}

                    echo "****** END : READING TECH USER ******"
                """

            cmdResult = sh script: cmd,returnStatus: true

            if (cmdResult == 0) {
            
                echo "[INFO] Stage 'Reading Tech User' SUCCESS."
            
            } else {

                errMessage = "[FAILED] : An unexpected error occured while reading your tech_user. Take a look at this <br>" 
                println errMessage
                sh "exit 1"

            }

            } else {

                println "The stage 'Getting Jenkins Env Vars' was skipped"
            }
        }
    }
```

**AMAZING !!!!! CONGRATULATION Les DesCodeuses !!! Well done you're a Jenkins Warrior now**
