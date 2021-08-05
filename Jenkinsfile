def buildDir = 'build/'
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Prepare') {
            steps {
                // remove build directories
                sh "rm -rf ${buildDir}api"
                sh "rm -rf ${buildDir}code-browser"
                sh "rm -rf ${buildDir}coverage"
                sh "rm -rf ${buildDir}logs"
                sh "rm -rf ${buildDir}pdepend"
                // create build directories
                sh "mkdir -p ${buildDir}api"
                sh "mkdir -p ${buildDir}code-browser"
                sh "mkdir -p ${buildDir}coverage"
                sh "mkdir -p ${buildDir}logs"
                sh "mkdir -p ${buildDir}pdepend"
                // set permission
                sh "chmod -R 777 ./storage/"
                sh "chmod -R 777 ./bootstrap/cache/"
            }
        }
        stage('Environment') {
            steps {
                // set production environment
                sh 'mv .env.production .env'
                sh 'rm -f .env.*'
            }
        }
        stage('Composer') {
            steps {
                sh 'composer install --no-suggest --optimize-autoloader --no-ansi --no-interaction'
            }
        }
        stage('Yarn & Webpack') {
            steps {
                sh 'yarn'
                sh 'yarn run production'
            }
        }
        stage('PHPUnit Tests') {
            steps {
                sh "phpunit -c phpunit.xml --coverage-html ${buildDir}coverage --coverage-clover ${buildDir}coverage/clover.xml --coverage-crap4j ${buildDir}logs/crap4j.xml --log-junit ${buildDir}logs/junit.xml"
            }
        }
        stage('Code Tests and Documentation') {
            steps {
                // code sniffer
                sh "phpcs --report=checkstyle --report-file=${buildDir}logs/checkstyle.xml --standard=${buildDir}phpcs.xml --extensions=php,inc --ignore=autoload.php --ignore=vendor/ app || exit 0"
                // mess detector
                sh "phpmd app xml ${buildDir}phpmd.xml --reportfile ${buildDir}logs/pmd.xml --exclude vendor/ --exclude autoload.php || exit 0"
                // copy&paste detector
                sh "phpcpd --log-pmd ${buildDir}logs/pmd-cpd.xml --exclude vendor app || exit 0"
                // lines of code
                sh "phploc --count-tests --exclude vendor/ --log-csv ${buildDir}logs/phploc.csv --log-xml ${buildDir}logs/phploc.xml app"
                // software metrics
                sh "pdepend --jdepend-xml=${buildDir}logs/jdepend.xml --jdepend-chart=${buildDir}pdepend/dependencies.svg --overview-pyramid=${buildDir}pdepend/overview-pyramid.svg --ignore=vendor app"
            }
        }
        stage('Publish Reporting') {
            steps {
                junit "${buildDir}logs/junit.xml"
                publishHTML (target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: "${buildDir}coverage",
                    reportFiles: 'index.html',
                    reportName: "PHP Code Coverage"
                ])
                recordIssues enabledForFailure: true, tool: checkStyle(pattern: "${buildDir}logs/checkstyle.xml")
                recordIssues enabledForFailure: true, tool: pmdParser(pattern: "${buildDir}logs/pmd.xml")
                recordIssues enabledForFailure: true, tool: cpd(pattern: "${buildDir}logs/pmd-cpd.xml")
                sh "echo '<html><head><title>PHP Dependency</title></head><body></body><img src=\"dependencies.svg\"></br><img src=\"overview-pyramid.svg\"></html>' > ${buildDir}pdepend/index.html"
                publishHTML (target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: "${buildDir}pdepend",
                    reportFiles: 'index.html',
                    reportName: "PHP Dependencies"
                ])
            }
        }
        stage('Deployment') {
            steps {
                sh "rsync -avz --delete --exclude-from=.deployment_exclude -e 'ssh -p 2530' . deploy@www.example.com:/www/example-project/"
            }
        }
    }
}