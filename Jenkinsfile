pipeline{
    
    agent {
        label 'dockerslave'
    }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
        terraform 'Terraform'
    }

    environment {
        // IMAGE = "panda"
        // VERSION = "0.0.1"
        ANSIBLE = tool name: 'Ansible', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    
    stages{
        stage("Clear running apps"){
            steps {
                echo "CLEAN"
                sh 'docker rm -f pandaapp || true'
            }
        }
         stage("Get code"){
            steps {
                echo "CODE"
                git branch: 'master', url: 'https://github.com/Awioniks/test_git_panda.git'
            }
        }
         stage("Build and Junit"){
            steps {
                echo "BUILD"
                sh "mvn -Dmaven.test.failure.ignore=true clean package"

            }
        }
         stage("Build Docker iamge"){
            steps {
                echo "IMAGE"
                sh "mvn package -Pdocker"
                
            }
        }
         stage("Run docker app"){
            steps {
                echo "RUN DOCKER"
                sh "docker run -d -p 8080:8080 --name pandaapp ${IMAGE}:${VERSION}"
            }
        }
         // stage("Test Selenium"){
            // steps {
                // echo "TEST SELENIUM"
                // sh "mvn -Pselenium test"

            // }
        // }
        stage("Deploy jar to artifactory"){
            steps {
                echo "Deploy"
                configFileProvider([configFile(fileId: '4952391e-e5ff-4b1c-bd5f-a5bc133800ea', variable: 'maven_settings')]) {
                    sh "mvn -s $maven_settings deploy"
                }
            }
            post {
                always {
                    sh 'docker stop pandaapp'
                    deleteDir()
                }
            }
        }
        stage('Test installation') {
            steps {
                sh 'terraform --version'
                sh 'ansible --version'
            }
        }
        stage('Check AWS Env') {
            steps {
                    dir('infrastucture/terraform'){
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-id']]) {
                        sh "infrastructure init && terraform apply -auto-approve -var-file panda.tfvars"
                    }
                    withCredentials([file(credentialsId: 'aws_keys', variable: 'pandafile')]) {
                        sh "cp \$pandafile ../panda.pem"
                    }   
                }
            }
        }
        stage('Copy Ansible role') {
            steps {
                sh 'cp infrastructure/ansible/panda /etc/ansible/roles'
            }
        }
        stage('Run Ansible') {
            steps {
                dir('infrastructure/ansible')
                sh 'chmod 600 ../panda.pem'
                sh 'ansible-playbook -i ./inventory playbook.yml'
            }
        }
    }
}
