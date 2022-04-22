node{


        def docker
        def dockerCMD
        def tagName = "1.0"

        stage('preparation'){
             echo "preparing the environment..."
             docker=tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
             dockerCMD = "$docker/docker"
        }

        stage ('P1 - Job1 - Install and Configure Puppet - Build'){
            try{
                echo "Installing Puppet Agent..."
                node('SlaveNode') {
                    sh "sudo /opt/puppetlabs/bin/puppet agent -t"
                }
            }
            catch(Exception e) {
                echo "Exception occurred when on JOB 1"
                currentBuild.result="FAILURE"
                throw e
            }
        }


        stage ('P1 - Job2 - Ansible Configuration '){
            try{
                echo "Checkout Repo on SlaveNode to get the playbook.yml file..."
                node ('SlaveNode') {
 	                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/llorenckaferocks/edureka-project1']]])
                }

                echo "Applying Ansible Configuration..."
                node ('SlaveNode') {
                    ansiblePlaybook disableHostKeyChecking: true, colorized: false, installation: 'Ansible', inventory: '', playbook: 'playbook.yml', sudoUser: null
                }
            }
            catch(Exception e) {
                 echo "Exception occurred on JOB 2"
                 currentBuild.result="FAILURE"
                 throw e
            }

        }

        stage('P1 - Job3 - Deploy Application on Test Server'){
            try{
                echo "Building docker image for application..."
                sh "${dockerCMD} build -t llorenckaferocks/edureka-project1:${tagName} ."

                echo "Pushing docker image to DockerHub...."
                withDockerRegistry([ credentialsId: "dockerHubCredentials", url: "" ])  {
                    sh "${dockerCMD} push llorenckaferocks/edureka-project1:${tagName}"
                }

                echo "Removing Old Container..."
                node ('SlaveNode') {
                    ansiblePlaybook disableHostKeyChecking: true, colorized: false, installation: 'Ansible', inventory: '', playbook: 'remove-container-playbook.yml', sudoUser: null
                }

                echo "Deploying Container..."
                node ('SlaveNode') {
                    ansiblePlaybook disableHostKeyChecking: true, colorized: false, installation: 'Ansible', inventory: '', playbook: 'deploy-container-playbook.yml', sudoUser: null
                }
            }
            catch(Exception e) {
                 echo "Exception occurred on JOB 3"
                 currentBuild.result="FAILURE"
                 throw e
            }
            finally{
                def currentResult = currentBuild.result ?: 'SUCCESS'
                if (currentResult == 'FAILURE') {
                    echo "Removing Old Container..."
                    node ('SlaveNode') {
                        ansiblePlaybook disableHostKeyChecking: true, colorized: false, installation: 'Ansible', inventory: '', playbook: 'remove-container-playbook.yml', sudoUser: null
                    }
                }
            }
        }
}

