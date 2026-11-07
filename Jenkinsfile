pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/ThisIsMikeyS/cicd-demo-app.git', description: 'Ссылка на Fork')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Ветка для сборки')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Выполнять unit-тесты')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${params.REPO_URL}", branch: "${params.GIT_BRANCH}"
                script {
                    def sha = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Ветка: ${params.GIT_BRANCH}"
                    echo "Полный SHA коммита: ${sha}"
                    env.GIT_COMMIT_SHA = sha
                }
            }
        }

        stage('Validate') {
            steps {
                script {
                    def missing = []
                    if (!fileExists('README.md'))        missing << 'README.md'
                    if (!fileExists('requirements.txt')) missing << 'requirements.txt'
                    if (!fileExists('src'))              missing << 'src/'
                    if (!fileExists('tests'))            missing << 'tests/'
                    if (missing) {
                        error "Отсутствуют обязательные элементы: ${missing.join(', ')}"
                    }
                    echo 'Структура проекта в порядке.'
                }
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Build number: ${BUILD_NUMBER}" > build-info.txt
                    echo "Commit SHA: ${GIT_COMMIT_SHA}" >> build-info.txt
                    tar -czf app.tar.gz src tests README.md requirements.txt build-info.txt
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Сборка успешно завершена.'
        }
        failure {
            echo 'Сборка завершилась с ошибкой.'
        }
    }
}
