# poc-scripts

### Scripts for IO Release: 2021.12.4

| Script | Purpose |
| ---| --- |
| IO-Prescription-Workflow | Base Jenkinsfile that gets the IO prescription & runs the workflow stage |
| IO-Prescription-Polaris-Workflow | Jenkinsfile that runs the Polaris satge in addition to the above |
| IO-Prescription-Polaris-BlackDuck-Workflow | Jenkinsfile that runs Black Duck in addition to the above |
| IO-External-Prescription-Workflow | Jenkinsfile that uses SCM type as 'External' to get the IO prescription |
| prescription.sh | Prescription shell-script for 2021.12.4 (adds `codePatch` in the API call to IO IQ to support SCM type of 'external') |
| io-manifest.json | Sample IO Manifest JSON (includes `codePatch` parameter to support SCM type of 'external') |

### Usage (SCM Type: Supported SCM)

* Commit the script directly to the repository as the Jenkinsfile.
* Use as the pipeline script directly in a Jenkins job configuration.

### Usage (SCM Type: External)

* Same as above, but the updated io-manifest.json from this branch needs to be included on the code repository.
