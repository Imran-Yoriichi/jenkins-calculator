pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        PYTHON      = 'python'
        APP_NAME    = 'Python Calculator'
        REPORT_DIR  = 'reports'
    }

    parameters {
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run the test suite?'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
                echo "Checked out branch: ${env.GIT_BRANCH}"
                echo "Commit: ${env.GIT_COMMIT}"
            }
        }

        stage('Setup') {
            steps {
                echo "Setting up Python environment for ${env.APP_NAME}..."
                bat 'python --version'
                bat 'pip install -r requirements.txt'
                bat 'mkdir reports 2>nul || echo Reports directory ready'
            }
        }

        stage('Lint') {
            steps {
                echo 'Checking code style...'
                bat 'pip install pyflakes'
                bat 'python -m pyflakes calculator.py'
                echo 'Lint passed — no syntax errors!'
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo 'Running test suite with pytest...'
                bat 'python -m pytest test_calculator.py -v --html=reports/test-report.html --self-contained-html'
            }
            post {
                always {
                    echo 'Test stage finished.'
                }
                success {
                    echo 'All tests passed!'
                }
                failure {
                    echo 'Some tests failed — check the report!'
                }
            }
        }

        stage('Archive Results') {
            steps {
                echo 'Archiving test report...'
                archiveArtifacts artifacts: 'reports/*.html', allowEmptyArchive: true
                echo 'Report archived — find it in the build artifacts!'
            }
        }

    }

    post {
        success {
            echo "Build ${env.BUILD_NUMBER} of ${env.APP_NAME} passed all checks!"
        }
        failure {
            echo "Build ${env.BUILD_NUMBER} of ${env.APP_NAME} FAILED — check console output!"
        }
        always {
            cleanWs()
            echo 'Workspace cleaned.'
        }
    }
}
