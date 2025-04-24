

# Steps to reproduce bug in Batch-mode export variables

---

Preparation to run the scripts


 1. In all JSON files in this folder, replace 111111 with your AWS Account id

 2. All resources are created in us-west-2 region

 3. Make sure "region" is set as "us-west-2" in you AWS Credentials file


---

Steps


1. `aws codestar-connections create-connection --provider-type GitHub --connection-name github-connection`

   Save the connection ARN in the output

2. Replace the variable "<CODE_CONNECTION_ARN>" in codebuildproj_batch.json, codebuildproj_single.json and pipeline1.json with the ARN above

3. Replace Aws Account Id in the command below and run it. 

   `aws s3 mb s3://codepipeline-us-west-2-<ACCOUNT_ID>`

4. `aws iam create-role --role-name CodeBuildServiceRole  --assume-role-policy-document file://codebuild-trust-policy.json`

5. `aws iam put-role-policy --role-name CodeBuildServiceRole --policy-name codebuildrole --policy-document file://codebuild-role.json`

6. `aws iam create-role --role-name CodePipelineServiceRole  --assume-role-policy-document file://codepipeline-trust-policy.json` 

7. `aws iam put-role-policy --role-name CodePipelineServiceRole --policy-name codepipelinerole  --policy-document file://codepipeline-role.json`

8. `aws codebuild create-project --cli-input-json file://codebuildproj_single.json`

9. `aws codebuild create-project --cli-input-json file://codebuildproj_batch.json`

10. `aws codebuild create-project --cli-input-json file://codedeployproj.json`

11. `aws codepipeline  create-pipeline  --cli-input-json file://pipeline1.json`


---

  The above pipeline creates two CodeBuild projects. 

  BuildProjSingle1 - with a single stage build file (https://github.com/doitintl/aws-codebuild-sample/blob/main/buildspec_single.yml) 
  BuildProjBatch1 - with a batch build file (https://github.com/doitintl/aws-codebuild-sample/blob/main/buildspec_batch.yml) 

  Both of them export an enviroment variable EXPORT_VAR1, meant to be used in subsequent stages

  CodePipeline is created with BuildProjSingle1 as the Build stage and a dummy deploy stage, which just outputs the environment variable.

  After the above steps, codepipeline should run without errors. Notice in the "Input section" of the Deploy stage, you will notice that the Environment variables are properly substituted from the outout of the Build stage.

  In the build-stage, this value of "EXPORT_VAR1" is set inside the code - https://github.com/doitintl/aws-codebuild-sample/blob/main/buildspec_single.yml


  Now, edit the Codepipeline so that Build stage will point to BuildProjBatch1. Make sure you select the "Batch Build" for "Build Type" when you make this change.

  The pipeline will now fail with "An action in this pipeline failed because one or more variables could not be resolved: Action name=Deploy. This can result when a variable is referenced that does not exist. Validate the configuration for this action."

  You can also notice that in the "Input section" of the Deploy stage, environment variables are not properly substituted.





