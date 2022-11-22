# tf-ci-module
This Terraform module is intended to create an AWS CodeCommit repository integrated with AWS CodePipeline and CodeBuild. The CodePipeline consists of two stages: 

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

### Notes about CodeCommit
1. Keep in mind that new repositories are **not** created with a default branch. Therefore, once the module has run, you must clone the repository, add a file, and then push to `origin/<repository_branch>` to initialize the repository. 


2. To use ssh for your git, you need to have an ssh key for code commit within your user's security credentials. For example, to authenticate automatically against all CodeCommit repositories, you can set up the Host within your ssh config like this: 

```
Host git-codecommit.*.amazonaws.com
  User SSH-KEY-ID (THIS IS NOT THE ACCESS SECRET KEY ID)
  IdentityFile ~/.ssh/YOURSSHKEY
  IdentitiesOnly yes
```

### Further notes
This module was written to enable the fast and repeatable creation of the CodeCommit, CodePipeline, CodeBuild Stack, and an Elastic Container Repository to push the previously built container.
