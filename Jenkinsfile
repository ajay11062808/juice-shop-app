pipeline{
  agent any
  tools{
    jdk 'Java17'
    maven 'Maven3'
    nodejs 'NodeJS20'
  }
  
  parameters {
        string(name: 'application_url', defaultValue: 'http://10.0.0.4:3000', description: 'URL for ZAP attack')
    }
  
  stages{
    stage('Cleanup Workspace'){
      steps{
        cleanWs()
          }
       }
     stage('Checkout from SCM')
    {
      steps{
          checkout scmGit(branches: [[name: '${ghprbActualCommit}']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', name: 'origin', refspec: '+refs/pull/${ghprbPullId}/*:refs/remotes/origin/pr/${ghprbPullId}/*', url: 'https://github.com/ajay11062808/juice-shop-app.git']])
          }
    }
    stage('Owasp Dependency Check'){
      steps{
        script{dependencyCheck additionalArguments: '--format HTML', odcInstallation: 'DP-check'}
      }
    }
     
    stage('Grype') {
                    steps {
                        sh 'grype dir:. -o json --file grype_result.json --scope AllLayers'
                     }
                }
        stage('Grype results to DefectDojo'){
            steps{
                script{
                    def buildNumber=currentBuild.getNumber()
                    def timestamp= new Date().format("dd-MM-yyyy_HH:mm:ss",TimeZone.getTimeZone("Asia/Kolkata"))
                    def engagementName="Engagement-${buildNumber}_Timestamp-${timestamp}"
                    env.ENGAGEMENT_NAME=engagementName
                    echo "Engagement name:${engagementName}"
                    sh "curl -X 'POST' \
  'http://10.0.0.4:8080/api/v2/reimport-scan/' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -H 'Authorization: Token 4f1944ca5dcf71096685b3364b7bbe028f8ddaff' \
  -H 'X-CSRFTOKEN: gFg5uD1zOJuWiKHAT60tB6F4kbUCzEsoQ8QMDGPzPQPUFai1E1Qa7e59v4HfrrZh' \
  -F 'product_type_name=' \
  -F 'active=true' \
  -F 'do_not_reactivate=false' \
  -F 'endpoint_to_add=' \
  -F 'verified=true' \
  -F 'close_old_findings=true' \
  -F 'test_title=' \
  -F 'engagement_name=${engagementName}' \
  -F 'build_id=${env.BUILD_ID}' \
  -F 'deduplication_on_engagement=' \
  -F 'push_to_jira=false' \
  -F 'minimum_severity=Low' \
  -F 'close_old_findings_product_scope=false' \
  -F 'scan_date=' \
  -F 'create_finding_groups_for_all_findings=true' \
  -F 'engagement_end_date=' \
  -F 'test=' \
  -F 'environment=' \
  -F 'service=' \
  -F 'commit_hash=' \
  -F 'group_by=' \
  -F 'version=' \
  -F 'tags=new tag' \
  -F 'api_scan_configuration=' \
  -F 'product_name=Juice shop' \
  -F 'file=@grype_result.json;type=application/json' \
  -F 'auto_create_context=true' \
  -F 'lead=' \
  -F 'scan_type=Anchore Grype' \
  -F 'branch_tag=' \
  -F 'source_code_management_uri='"
                }
            }
        }
     stage('Semgrep Analysis') {
          steps {
                     sh '''
                       pwd
                       sudo semgrep scan --config auto --output Semgrep_results.json --json
                       ''' 
            }
        }
    stage('Semgrep results to DefectDojo'){
            steps{
                script{
                    def engagementName=env.ENGAGEMENT_NAME
                    echo "Engagement name:${engagementName}"
                    sh "curl -X 'POST' \
  'http://10.0.0.4:8080/api/v2/reimport-scan/' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -H 'Authorization: Token 4f1944ca5dcf71096685b3364b7bbe028f8ddaff' \
  -H 'X-CSRFTOKEN: gFg5uD1zOJuWiKHAT60tB6F4kbUCzEsoQ8QMDGPzPQPUFai1E1Qa7e59v4HfrrZh' \
  -F 'product_type_name=' \
  -F 'active=true' \
  -F 'do_not_reactivate=false' \
  -F 'endpoint_to_add=' \
  -F 'verified=true' \
  -F 'close_old_findings=true' \
  -F 'test_title=' \
  -F 'engagement_name=${engagementName}' \
  -F 'build_id=${env.BUILD_ID}' \
  -F 'deduplication_on_engagement=' \
  -F 'push_to_jira=false' \
  -F 'minimum_severity=Low' \
  -F 'close_old_findings_product_scope=false' \
  -F 'scan_date=' \
  -F 'create_finding_groups_for_all_findings=true' \
  -F 'engagement_end_date=' \
  -F 'test=' \
  -F 'environment=' \
  -F 'service=' \
  -F 'commit_hash=' \
  -F 'group_by=' \
  -F 'version=' \
  -F 'tags=new tag' \
  -F 'api_scan_configuration=' \
  -F 'product_name=Juice shop' \
  -F 'file=@Semgrep_results.json;type=application/json' \
  -F 'auto_create_context=true' \
  -F 'lead=' \
  -F 'scan_type=Semgrep JSON Report' \
  -F 'branch_tag=' \
  -F 'source_code_management_uri='"
                }
            }
        }
    stage('SonarQube Analysis') {
    steps {
        script {
            // Set up the SonarQube environment
              def scannerHome = tool 'sonarqube-scanner'
              env.PATH = "${scannerHome}/bin:${env.PATH}"
            withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                      sh "sonar-scanner -Dsonar.projectKey=Juice-Shop-project"
            }
        }
    }
}
    stage('Quality Gate') {
            steps {
                script {
                    // Wait for the SonarQube analysis to complete and check the quality gate
                    waitForQualityGate abortPipeline: false,credentialsId:'jenkins-sonarqube-token'
                }
            }
        }
            stage('Checkout and Build Juice Shop') {
            steps {
                // Clone the Juice Shop repository
                checkout scmGit(branches: [[name: '${ghprbActualCommit}']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', name: 'origin', refspec: '+refs/pull/${ghprbPullId}/*:refs/remotes/origin/pr/${ghprbPullId}/*', url: 'https://github.com/ajay11062808/juice-shop-app.git']])

                // Go into the cloned folder
                dir('juice-shop-app') {
                    // Install dependencies
                    sh 'npm install'
                }
            }
        }

        stage('Start Juice Shop') {
            steps {
                // Start Juice Shop
                dir('juice-shop-app') {
                    sh 'npm start &'
                    sleep 20 // Give some time for the application to start (adjust as needed)
                }
            }
        }

        stage('Run OWASP ZAP Scan') {
          steps {
                script {
                  //Pull the latest Zap image from docker
                sh 'docker pull ghcr.io/zaproxy/zaproxy:stable'
                 // Run ZAP in a Docker container
                sh  'docker run -u root -v zap-data:/zap/wrk/:rw -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py -t ${application_url} -g gen.conf -r report.html || true'

            // Create a container from the ZAP Docker image to access the volume
            def zapContainer = docker.image('ghcr.io/zaproxy/zaproxy:stable').run("-v zap-data:/zap/wrk")
            // Copy the ZAP reports from the Docker volume to the Jenkins workspace
            sh "docker cp ${zapContainer.id}:/zap/wrk/report.html ." 
            sh "docker cp ${zapContainer.id}:/zap/wrk/gen.conf ."
            
            // Stop and remove the container
            zapContainer.stop()
                    
                }
            }
        }
            
 }
  post {
    success {
        echo "Pipeline currentResult: ${currentBuild.currentResult}"
        archiveArtifacts artifacts: 'report.html', fingerprint: true
    }
}
  
}
