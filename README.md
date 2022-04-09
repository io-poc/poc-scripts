# poc-scripts

### Scripts for IO Release: 2021.12.4

| Script | Purpose |
| ---| --- |
| IO-Prescription-Workflow | Base Jenkinsfile that gets the IO prescription & runs the workflow stage |
| IO-Prescription-Polaris-Workflow | Jenkinsfile that runs the Polaris satge in addition to the above |
| IO-Prescription-Polaris-BlackDuck-Workflow | Jenkinsfile that runs Black Duck in addition to the above |
| IO-External-Prescription-Workflow | Jenkinsfile that uses SCM type as 'External' to get the IO prescription |


### Usage

* Commit the script directly to the repository as the Jenkinsfile.
* Use as the pipeline script directly in a Jenkins job configuration.
