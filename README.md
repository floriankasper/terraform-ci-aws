# tf-ci-module
This Terraform module is intended to create an AWS CodeCommit repository integrated with AWS CodePipeline and CodeBuild[^1]. The CodePipeline consists of two stages: 

1. A Source stage that pulls from the CodeCommit repository
2. A Build stage that uses the artifacts of the Source Stage and executes commands from the `buildspec.yml` file.

The last step of the pipeline is publishing the docker container to a private Elastic Container Repository.

## Usage
```hcl
module "ci-module" {
    source                 = "git::https://github.com/floriankasper/tf-ci-module.git"
    repository_name           = "repository"              
    repository_branch         = "master"                     
    aws_account               = "000000000000"
    aws_region                = "eu-central-1"               
    delimiter                 = "-"                         
    environment               = "dev"                        
    buildspecs                = "buildspec.yml"  
    build_compute_type        = "BUILD_GENERAL1_SMALL"               
    build_image               = "aws/codebuild/standard:6.0"
    build_timeout             = "10"    
    build_privileged_override = "true"                                
    concurrent_builds         = "1"                  
}
````

## Notes
### CodeCommit
1. Keep in mind that new repositories are **not** created with a default branch. Therefore, once the module has run, you must clone the repository, add a file, and then push to `origin/<repository_branch>` to initialize the repository. 
    ```
    git init
    git remote add origin ssh://git-codecommit.eu-central-1.amazonaws.com/v1/repos/your-repo
    git add .
    git commit -m "My first commit"
    git push --set-upstream origin master
    ```

2. To use ssh, you need to have an ssh key for code commit within your user's security credentials; if you prefer https, look [here](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html). For example, if you like to create a dedicated user for Codecommit, you could use this snippet within your main.tf file.
    ```hcl
    resource "aws_iam_user" "git_user" {
      name      = "codecommit_user"
    }

    resource "aws_iam_user_ssh_key" "git_user" {
      username   = aws_iam_user.git_user.name
      encoding   = "SSH"
      public_key = "ssh-rsa KEY"
    }
    ```

3. To authenticate automatically against all CodeCommit repositories via ssh, you can set up the Host within your ssh config like this: 

    ```
    Host git-codecommit.*.amazonaws.com
      User SSH-KEY-ID (THIS IS NOT THE ACCESS KEY ID)
      IdentityFile ~/.ssh/YOURSSHKEY
      IdentitiesOnly yes
    ```


[^1]: This module was written to enable the fast and repeatable creation of the CodeCommit, CodePipeline, CodeBuild Stack, and an Elastic Container Repository to push the previously built container.
