pipeline {
    agent any

    environment {
        //
        mainRepoName = "lkmodel"
        //
        // currentRepoName = "${GIT_URL.substring(GIT_URL.lastIndexOf('/')+1, GIT_URL.length()-4)}"
        // NODE_BASE_NAME = "ui-node-${GIT_COMMIT.substring(0, 6)}"
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_lkmodel"
        REPORT_PATH = "allure"
        GITHUB_URL_PREFIX = "https://github.com/kraigyang/"
        GITHUB_URL_SUFFIX = ".git"
        //
        buildNumber = "${currentBuild.number}"
        //
        allureReportUrl = "${JENKINS_URL}/${JOB_PATH}/${buildNumber}/${REPORT_PATH}"
        FROM_EMAIL="bityk@163.com"
        REPORT_EMAIL="528198540@qq.com"
        //
        // GA_TOKEN = credentials('GithubAccessLongTerm')
        // GA_REPO_OWNER = 'kraigyang'
        // GA_REPO_NAME = "${currentRepoName}"
        //
        // GA_COMMIT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
    }
    
    stages {
        stage("multirepos-ci") {
            steps {
                script {
		        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
	                    sh "rm -rf $WORKSPACE/report"
	                    parallel repoJobs()
			}
                }
            }
        }
        stage("reports-merge"){
            steps {
                script {
                    echo "-------------------------allure report generating start---------------------------------------------------"
                    sh 'cd $WORKSPACE && allure generate ./report/result -o ./report/html --clean'
                    allure includeProperties: false, jdk: 'jdk17', report: "report/html", results: [[path: "report/result"]]
                    echo "-------------------------allure report generating end ----------------------------------------------------"
                }
            }
        } 
    }
}

def repos() {
  return ["$mainRepoName"]
}

def repoJobs() {
  jobs = [:]
  repos().each { repo ->
    jobs[repo] = { 
        stage(repo + "checkout"){
           echo "$repo checkout"
           sh "rm -rf  $repo; git clone $GITHUB_URL_PREFIX$repo$GITHUB_URL_SUFFIX; echo `pwd`;"
        }
        stage(repo + "autotesting"){
                script {
		        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                           withEnv(["repoName=$repo"]) { // it can override any env variable
                                echo "repoName = ${repoName}"
                                echo "$repo autotesting"
                                sh 'printenv'
                                sh "cp -r /home/jenkins_home/pytest $WORKSPACE/$repo"
                                echo "--------------------------------------------$repo test start------------------------------------------------"
                                if (repoName == mainRepoName){
                                  sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m mainrepo --cmdrepo=${repoName} -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                                } else {
                                  sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m childrepo --cmdrepo=${repoName} -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                                }
                                echo "--------------------------------------------$repo test end  ------------------------------------------------"
                          }
		       }
               }
        }
        stage(repo + "reports"){
            withEnv(["repoName=$repo"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repo reports"
                //
                echo "$repo Allure Report URL: ${allureReportUrl}"
                echo "-------------------------$repo allure report generating start---------------------------------------------------"
                sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && cp -r ./report $WORKSPACE'
                echo "-------------------------$repo allure report generating end ----------------------------------------------------"
            }
        }
    }
  }
  return jobs
}
