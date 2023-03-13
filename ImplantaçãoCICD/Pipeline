pipeline {
    agent any
    
    environment {
        SERVER  = '' 
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: '',
                    credentialsId: "",
                    branch: ("${env.BRANCH}".split("/").length > 3 ? "${env.BRANCH}".split("/")[2] + "/" + "${env.BRANCH}".split("/")[3] : "${env.BRANCH}".split("/")[2])
            }
        }
        stage ("Install Packages with Yarn") {
            steps {
                powershell script: """
                    \$ErrorActionPreference = 'Stop'
                    yarn
                """
            }
        }
        stage ("Build Application with Yarn") {
            steps {
                powershell script: """
                    \$ErrorActionPreference = 'Stop'
                    yarn build
                """
            }
        }
        stage ("Compress files") {
            steps {
                powershell script: """
                    \$ErrorActionPreference = 'Stop'
                    Compress-Archive -Path "$WORKSPACE\\build\\*" -DestinationPath "$WORKSPACE\\OSBMobile.zip" -Force
                """
            }
        }
        stage ("Send files into server") {
            steps {
                powershell script: """
                    \$ErrorActionPreference = 'Stop'
                    Copy-Item -Path "$WORKSPACE\\OSBMobile.zip" `
                        -Destination "\\\\$SERVER\\c\$\\zips para extracao" -Force
                """
            }
        }
        stage('Uncompress Files Into Server'){
            steps{
                powershell script: """
                    \$ErrorActionPreference = 'Stop'
                    \$USER = '$env.USER'
                    \$PASS = '$env.PASS'
                    \$SERVER = '${SERVER}'
                    \$PASSWORD = \$PASS | ConvertTo-SecureString -AsPlainText -Force
                    \$CRED = New-Object System.Management.Automation.PSCredential -ArgumentList \$USER, \$PASSWORD

                    Invoke-Command -ComputerName \$SERVER -Credential \$CRED -ScriptBlock {
                        if(-Not (Test-Path -Path "C:\\zips para extracao\\OSBMobile")){
                            mkdir "C:\\zips para extracao\\OSBMobile" -Force
                        }

                        Remove-Item C:\\FitBank\\Deploy\\Sites\\OSBMobile\\static\\* -Recurse -Force -Verbose

                        Expand-Archive -Path "C:\\zips para extracao\\OSBMobile.zip" -DestinationPath "C:\\zips para extracao\\OSBMobile" -Force
                        Copy-Item -Path "C:\\zips para extracao\\OSBMobile\\*" -Destination C:\\FitBank\\Deploy\\Sites\\OSBMobile  -Recurse -Force -Verbose

                        Remove-Item "C:\\zips para extracao\\OSBMobile" -Recurse -Force
                    }
                """
                }
            }
        }
    post {
        //always {
            //echo 'Realizando a limpeza do Workspace!'
        //}
        success {
            echo 'Deploy com sucesso!'
        }
        failure {
            echo "Falha no deploy."
        }
    }
}
