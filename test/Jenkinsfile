#!groovy
properties([
  parameters([
    choice(
      name:"mutability",
      choices:"mutable\nimmutable",
      description:"Is this image mutable or immutable?"
    ),
    choice(
      name:"Select_env",
      choices:"dev\npreprod",
      description:"Which kind of image do you want?"
    ),
    string(
       name:"GCP_PROJECT_ID",
       defaultValue: "hsbc-8180904-gsna-dev"
       description: "Which GCP project to create the image in"
       )
  ]),
  pipelineTriggers([
    //run everyday at 1am UTC
    cron("H 01 * * *"),
    //Poll Git every 20 minutes for changes
    pollSCM("H/20 * * *")
  ]),
  buildDiscarder(
    logRotator(
      artifactDaysToKeepStr:"",
      artifactNumToKeepStr:"",
      daysToKeepStr:"60",
      numToKeepStr:""
    )
   )
  ])

  timeout(time:600,unit:"SECONDS"){
    node("GSNA_JENKINS_SLAVES"){
      ws("Workspace/${env.JOB_NAME}".replace("2%f","_")){
        try {
          withCredentials([
             file(
               credentialsId:"gce-stage3-image-builder-${GCP_PROJECT_ID)}",
               variable:"GCP_SA_KEY"
             )
          ]){
          //Get a copy of our files on the node
          cheout scm

          //Stages
          stage("Build image"){
            def repo = "gce-stage3-proxy"

          //Build our image family name
          def imageFamily = "hsbc-${mutability}-rhel-7-${repo}-"

          //Maximum image name length is 63,so we need to workout
          //how many characters we have left for branch name.
          def endIndex=63

          def imageName="hsbc-rhel-7-${repo}-"
          endIndex -=imageName.length()
          mutability = "-"+mutability + "-v"
          endIndex -=mutability.length()

          //Build a timestamp for now
          def timestamp = (new Date()).format("yyyyMMddHHmmss",TimeZone("UTC"))
          endIndex -=timestamp.length()

          def branchName = env.BRANCH_NAME.replace("2%f","-").toLowerCase()

          //keep our index within range of the string
          endIndex -=timestamp.length() <endIndex ? branchName.length() :endIndex

          //Append the branch name,the mutability of the image and the timestamp
            imageName = imageName +branchName.substring(0,endIndex)
            imageName = imageName +mutability +timestamp

          //Append the branch name
          imageFamily = imageFamily + branchName.substring(0,endIndex)

          //Take any trailing dashes off
          while (imageFamily.endsWith("-")){
            imageFamily = imageFamily.substring(0,imageFamily.length -1)
          }

          echo "Running packer to build ${imageName}"
          sh(
            script:"packer build \
              -machine-readable \
              -timestamp-ui \
              -var-file=variables.json \
              -var 'image_name=${imagename}' \
              -var 'image_family=${imageFamily}' \
              -var 'gcp_project_id=${GCP_PROJECT_ID}' \
              -var 'account_file=${GCP_SA_KEY}' \
              build.jason
          )
        }
      }
      currentBuild.result='success'
      }catch(Exception e){
        currentBuild.result='FAILURE'
        echo.e.getMessage()
      }
    }
  }
}

          }
        }
      }
    }
  }