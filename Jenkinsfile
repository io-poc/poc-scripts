def buildBreakerStatus

pipeline {
    agent any

    environment {
        SCM_TYPE = "github"
        SCM_OWNER ="<OWNER>"
        SCM_BRANCH = "master"
        REPO_NAME = "<REPO>"
        IO_SERVER__URL = "https://<IO_URL>/api/ioiq"
        IO_ACCESS_TOKEN = "<TOKEN>" //credentials('io-token')
        IO_PERSONA = "devsecops"
        IO_PROJECT_NAME = "<IO_PROJECT>"
        IO_PROJECT_VERSION = "1.0"
        POLARIS_SERVER_URL = "<POLARIS_URL>"
        POLARIS_ACCESS_TOKEN = "<TOKEN>" //credentials('polaris-token')
        POLARIS_PROJECT_NAME = "<POLARIS_PROJECT_NAME>"
        GITHUB_USERNAME = "<GITHUB_USERNAME>"
        GTIHUB_ACCESS_TOKEN = "<TOKEN>" //credentials('github-token')
        IS_SAST_ENABLED = "false"
        IS_SCA_ENABLED = "false"
        IS_DAST_ENABLED = "false"
        IS_IMAGE_SCAN_ENABLED = "false"
        IS_SAST_MANUAL_ENABLED = "false"
        IS_DAST_MANUAL_ENABLED = "false"
        WORKFLOW_CLIENT_VERSION = "2021.12.4"
        WORKFLOW_URL = "https://<IO_URL>/api/workflowengine"
   }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Checkout Source Code'
                // git branch: 'master', url: '<GIT REPO>'
            }
        }

        stage('Build Source Code') {
            steps {
                echo 'Build Source Code'
                // sh 'mvn -e clean package -DskipTests'
            }
        }

        stage('IO - Prescription') {
            steps {
                echo "Getting IO Prescription"
                sh '''
                rm -fr prescription.sh
                wget "https://raw.githubusercontent.com/synopsys-sig/io-artifacts/${WORKFLOW_CLIENT_VERSION}/prescription.sh"
                sed -i -e 's/\r$//' prescription.sh
                chmod a+x prescription.sh
                ./prescription.sh \
                --stage="IO" \
                --persona=${IO_PERSONA} \
                --io.url="${IO_SERVER__URL}" \
                --io.token="${IO_ACCESS_TOKEN}" \
                --manifest.type="json" \
                --project.name="${IO_PROJECT_NAME}" \
                --workflow.url="${WORKFLOW_URL}" \
                --workflow.version="${WORKFLOW_CLIENT_VERSION}" \
                --scm.type=${SCM_TYPE} \
                --scm.owner=${SCM_OWNER} \
                --scm.repo.name=${REPO_NAME}\
                --scm.branch.name=${SCM_BRANCH} \
                --github.owner.name=${SCM_OWNER} \
                --github.repository.name=${REPO_NAME}\
                --github.username=${GITHUB_USERNAME} \
                --github.token="${GTIHUB_ACCESS_TOKEN}" \
                --polaris.project.name=${POLARIS_PROJECT_NAME} \
                --polaris.url="${POLARIS_SERVER_URL}" \
                --polaris.token="${POLARIS_ACCESS_TOKEN}" \
                --jira.enable="false" \
                --IS_SAST_ENABLED="${IS_SAST_ENABLED}" \
                --IS_SCA_ENABLED="${IS_SCA_ENABLED}" \
                --IS_DAST_ENABLED="${IS_DAST_ENABLED}"
                '''
                sh 'mv result.json io-presciption.json'
                sh '''
                echo "==================================== IO Risk Score =======================================" > io-risk-score.txt
                echo "Business Criticality Score - $(jq -r '.riskScoreCard.bizCriticalityScore' io-presciption.json)" >> io-risk-score.txt
                echo "Data Class Score - $(jq -r '.riskScoreCard.dataClassScore' io-presciption.json)" >> io-risk-score.txt
                echo "Access Score - $(jq -r '.riskScoreCard.accessScore' io-presciption.json)" >> io-risk-score.txt
                echo "Open Vulnerability Score - $(jq -r '.riskScoreCard.openVulnScore' io-presciption.json)" >> io-risk-score.txt
                echo "Change Significance Score - $(jq -r '.riskScoreCard.changeSignificanceScore' io-presciption.json)" >> io-risk-score.txt
                export bizScore=$(jq -r '.riskScoreCard.bizCriticalityScore' io-presciption.json | cut -d'/' -f2)
                export dataScore=$(jq -r '.riskScoreCard.dataClassScore' io-presciption.json | cut -d'/' -f2)
                export accessScore=$(jq -r '.riskScoreCard.accessScore' io-presciption.json | cut -d'/' -f2)
                export vulnScore=$(jq -r '.riskScoreCard.openVulnScore' io-presciption.json | cut -d'/' -f2)
                export changeScore=$(jq -r '.riskScoreCard.changeSignificanceScore' io-presciption.json | cut -d'/' -f2)
                echo -n "Total Score - " >> io-risk-score.txt && echo "$bizScore + $dataScore + $accessScore + $vulnScore + $changeScore" | bc >> io-risk-score.txt
                '''
                sh 'cat io-risk-score.txt'
                sh '''
                echo "IS_SAST_ENABLED = $(jq -r '.security.activities.sast.enabled' io-presciption.json)" > io-prescription.txt
                echo "IS_SCA_ENABLED = $(jq -r '.security.activities.sca.enabled' io-presciption.json)" >> io-prescription.txt
                echo "IS_DAST_ENABLED = $(jq -r '.security.activities.dast.enabled' io-presciption.json)" >> io-prescription.txt
                echo "IS_IMAGE_SCAN_ENABLED = $(jq -r '.security.activities.imageScan.enabled' io-presciption.json)" >> io-prescription.txt
                echo "IS_SAST_MANUAL_ENABLED = $(jq -r '.security.activities.sastplusm.enabled' io-presciption.json)" >> io-prescription.txt
                echo "IS_DAST_MANUAL_ENABLED = $(jq -r '.security.activities.dastplusm.enabled' io-presciption.json)" >> io-prescription.txt
                '''
                sh 'cat io-prescription.txt'
            }
        }

        stage('SAST - Polaris') {
            steps {
                echo 'Running SAST using Polaris'
                sh '''
                  IS_SAST_ENABLED=$(jq -r '.security.activities.sast.enabled' io-presciption.json)
                  echo "IS_SAST_ENABLED = ${IS_SAST_ENABLED}"
                  if [ ${IS_SAST_ENABLED} = "true" ]; then
                    echo "Running Coverity on Polaris based on IO Precription"
                    rm -fr /tmp/polaris
                    wget -q ${POLARIS_SERVER_URL}/api/tools/polaris_cli-linux64.zip
                    unzip -j polaris_cli-linux64.zip -d /tmp
                    /tmp/polaris --persist-config --co project.name="${POLARIS_PROJECT_NAME}" --co project.branch="main" --co capture.build.buildCommands="null" --co capture.build.cleanCommands="null" --co capture.fileSystem="null" --co capture.coverity.autoCapture="enable"  configure
                    /tmp/polaris analyze -w
                  else
                    echo "Skipping Coverity on Polaris based on IO Precription"
                  fi
                  '''
            }
        }

        stage('IO - Workflow') {
            steps {
                echo 'Execute Workflow Stage'

                script {
                    sh '''
                    IS_SAST_ENABLED=$(jq -r '.security.activities.sast.enabled' io-presciption.json)
                    IS_SCA_ENABLED=$(jq -r '.security.activities.sca.enabled' io-presciption.json)
                    ./prescription.sh \
                    --stage="WORKFLOW" \
                    --persona=${IO_PERSONA} \
                    --io.url="${IO_SERVER__URL}" \
                    --io.token="${IO_ACCESS_TOKEN}" \
                    --manifest.type="json" \
                    --project.name="${IO_PROJECT_NAME}" \
                    --workflow.url="${WORKFLOW_URL}" \
                    --workflow.version="${WORKFLOW_CLIENT_VERSION}" \
                    --scm.type=${SCM_TYPE} \
                    --scm.owner=${SCM_OWNER} \
                    --scm.repo.name=${REPO_NAME}\
                    --scm.branch.name=${SCM_BRANCH} \
                    --github.owner.name=${SCM_OWNER} \
                    --github.repository.name=${REPO_NAME}\
                    --github.username=${GITHUB_USERNAME} \
                    --github.token="${GTIHUB_ACCESS_TOKEN}" \
                    --polaris.project.name=${POLARIS_PROJECT_NAME} \
                    --polaris.url="${POLARIS_SERVER_URL}" \
                    --polaris.token="${POLARIS_ACCESS_TOKEN}" \
                    --jira.enable="false" \
                    --IS_SAST_ENABLED="${IS_SAST_ENABLED}" \
                    --IS_SCA_ENABLED="false" \
                    --IS_DAST_ENABLED="false"
                    '''
                    echo "Running IO Workflow Engine with CodeDx"
                    sh '''
                    java -jar WorkflowClient.jar --workflowengine.url="${WORKFLOW_URL}" --io.manifest.path=synopsys-io.json
                    '''

                    if (fileExists('wf-output.json')) {
                        def workflowJSON = readJSON file: 'wf-output.json'
                        buildBreakerStatus = workflowJSON.breaker.status

                        print("========================== IO WorkflowEngine Summary ============================")
                        print("Build Breaker Status: $buildBreakerStatus")

                        if(workflowJSON.summary.size() > 0) {
                            workflowJSON.summary.each{ activity->
                                print("Activity: ${activity.activity}")
                                if(activity.has("breakercount")) {
                                    breakerCount = activity.breakercount.size()
                                    print("Build Breaker Count: $breakerCount")
                                    if (breakerCount > 0) {
                                        activity.breakercount.each{ breaker ->
                                            print("Severity: ${breaker.severity}")
                                            print("Count: ${breaker.count}")
                                        }
                                    }
                                }
                                if(activity.has("risk_score")) {
                                    print("Code Dx Risk Score: ${activity.risk_score}")
                                }
                            }
                        } else {
                            print("No workflow summary available.")
                        }
                        print("========================== IO WorkflowEngine Summary ============================")
                    } else {
                        print("Workflow Engine output not available.")
                    }
                }
            }
        }

        stage('Security Sign-Off') {
            steps {
                script {
                    if (buildBreakerStatus) {
                        input message: 'One or more conditions triggered Build Breaker. Do you wish to proceed?'
                    }
                }
                echo "Security Sign-Off Check Complete (Approved or Not Applicable)"
            }
        }
    }

    // post {
    //     always {
    //         cleanWs()
    //     }
    // }
}
