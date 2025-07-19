pipeline {
    agent any

    environment {
        REPO_URL = "http://git-server:3000/max/python-app.git"
        REPO_DIR = "${WORKSPACE}/python-app"  // Use Jenkins workspace directory
        BRANCH_NAME = "main"
        RUN_TESTS = "true"
        PYTHON_VERSION = "python3"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "Cloning branch: ${BRANCH_NAME} from ${REPO_URL}"
                    sh """
                        if [ -d "${REPO_DIR}" ]; then
                            cd ${REPO_DIR}
                            git reset --hard
                            git checkout ${BRANCH_NAME}
                            git pull origin ${BRANCH_NAME}
                        else
                            git clone --branch ${BRANCH_NAME} ${REPO_URL} ${REPO_DIR}
                        fi
                    """
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing dependencies..."
                    dir("${REPO_DIR}") {
                        sh """
                            set -e
                            ${PYTHON_VERSION} -m venv venv
                            . venv/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt
                        """
                    }
                }
            }
        }

        stage('Run Tests') {
            when {
                expression { return env.RUN_TESTS == "true" }
            }
            steps {
                script {
                    echo "Running tests..."
                    dir("${REPO_DIR}") {
                        sh """
                            set -e
                            . venv/bin/activate
                            pytest --junitxml=reports/test-results.xml
                        """
                    }
                }
            }
        }

        stage('Run Application') {
            steps {
                script {
                    echo "Starting Flask app for 60 seconds..."
                    dir("${REPO_DIR}") {
                        sh '''
                            set -e
                            . venv/bin/activate
                            echo "Flask app will run for 60 seconds..."
                            timeout 60s python app.py || echo "App terminated after timeout, ignoring error"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
