node {
    stage('Checkout') {
        checkout scm

        checkout([
            $class: 'GitSCM',
            branches: [
                [
                    name: 'refs/remotes/origin/gerrit-refspec'
                ]
            ], 
            doGenerateSubmoduleConfigurations: false,
            extensions: [
                [
                    $class: 'RelativeTargetDirectory',
                    relativeTargetDir: 'module'
                ], 
                [
                    $class: 'CleanCheckout'
                ]
            ], 
            submoduleCfg: [],
            userRemoteConfigs: [
                [
                    refspec: '+$GERRIT_REFSPEC:refs/remotes/origin/gerrit-refspec', 
                    credentialsId: 'jenkins',
                    url: 'ssh://gerrit.chop-chop.org:29418/Doofinder-M1'
                ]
            ]
        ])
    }

    stage('Collect open tasks') {
        openTasks \
            canComputeNew: false,
            defaultEncoding: '',
            excludePattern: '',
            healthy: '',
            high: 'FIXME',
            ignoreCase: true,
            low: 'NOTICE',
            normal: 'TODO',
            pattern: 'module/**',
            unHealthy: ''
    }

    stage('Build') {
        withCredentials([string(credentialsId: 'composer-auth', variable: 'COMPOSER_AUTH')]) {
            sh 'ant -emacs build'
        }
    }

    stage('Lint') {
        sh 'ant -emacs lint'

        checkstyle \
            canComputeNew: false,
            defaultEncoding: '',
            failedTotalAll: '0',
            healthy: '',
            pattern: 'reports/**/*.checkstyle.xml',
            unHealthy: '',
            unstableTotalAll: '0'

        pmd \
            canComputeNew: false,
            defaultEncoding: '',
            failedTotalAll: '0',
            healthy: '',
            pattern: 'reports/**/*.pmd.xml',
            unHealthy: '',
            unstableTotalAll: '0'

        dry \
            canComputeNew: false,
            defaultEncoding: '',
            failedTotalAll: '0',
            healthy: '',
            pattern: 'reports/**/*.cpd.xml',
            unHealthy: '',
            unstableTotalAll: '0'
    }

    stage('Test') {
        try {
            sh 'cp .env.example .env'
            sh 'docker-compose -f docker-compose.yml build --pull'
            sh 'EXTERNAL_HTTP_PORT="" docker-compose -f docker-compose.yml up -d'
            sh 'docker-compose -f docker-compose.yml exec -T app bash -c \'dockerize -wait http://localhost -wait tcp://$MYSQL_HOST:3306 -timeout 60s && cd $APPLICATION_PATH && gosu $APPLICATION_USER ant -emacs test-functional\''
        } finally {
            sh 'docker-compose -f docker-compose.yml down -v --remove-orphans'
        }

        junit 'reports/behat/*.xml'
    }

    stage('Archive') {
        archiveArtifacts 'reports/**'
    }
}
