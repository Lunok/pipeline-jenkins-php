node {
    // BUILD PROJECT FROM ORIGIN
    gitlabBuilds(builds: ['Get code from SSH'])
    {
        stage('Get code from SSH')
        { 
            gitlabCommitStatus(name: 'Get code from SSH')
            {
                git branch: env.BRANCH_NAME, credentialsId: 'jenkins_token_gitlab', url: 'git@127.0.0.1:root/filrouge.git'
                sh 'git clean -xdf'
            }
        }
    }
    
    // INSTALL DEPENDENCIES WITH COMPOSER
    gitlabBuilds(builds: ['Composer install'])
    {
        stage('Composer Install')
        {
            gitlabCommitStatus(name: 'Composer install') {
                sh 'composer install'
            }
        }
    }
    
    /**
     * TEST STANDARDS SYNTAX
     * Linting (as a debugger)
     */
    gitlabBuilds(builds: ['PHPLint'])
    {
       stage("PHPLint")
       {
            gitlabCommitStatus(name: 'PHPLint')
            {
                def RESULT = sh returnStatus: true, script: 'vendor/bin/parallel-lint --exclude vendor --exclude  var --exclude assets --exclude docker --exclude templates --exclude node_modules --exclude src/Migrations --exclude config --exclude public --exclude autoload.php ./'
                 
                if ( RESULT != 0 )
                {
                    currentBuild.result = 'FAILURE'
                    error 'Des erreurs ont été détectées pour PHPLint : ${RESULT}'
                }
            }
        }
    }
    
    /**
     * TEST CODE QUALITY WITH PHPCS
     * Check syntax code (forget ;, forget ) etc...)
     */
    gitlabBuilds(builds: ['Checkstyle Report']) 
    {
        stage('Checkstyle Report')
        {
            gitlabCommitStatus(name: 'Checkstyle Report') 
            {
                def RESULT = sh returnStatus: true, script: 'vendor/bin/phpcs --report=checkstyle --report-file=./build/logs/phpcs.log.xml --standard=./phpcs.xml.dist --extensions=php,inc --ignore=./vendor,./var,./docker,./build,./assets,autoload.php,./public,./templates,./node_modules,./src/Migrations,./config ./'
                checkstyle defaultEncoding: 'UTF-8', pattern: 'build/logs/phpcs.log.xml'
                
                if ( RESULT != 0 ) 
                {
                    currentBuild.result = 'FAILURE'
                    error 'Des erreurs ont été détectées pour PHPCS : ${RESULT}'
                }
            }
        }
    }
    
    /*
     * MESS DETECTION REPORT
     * UNSED METHODS
     * UNSED PARAMETERS
     * UNSED PROPERTIES
     * CAMELCASE SYNTAX
     * TOO LONG NAME VARS/METHODS/PARAMETERS
     **/
    gitlabBuilds(builds: ['Mess detection'])
    {
        stage('Mess detection')
        {
            gitlabCommitStatus(name: 'Mess detection')
            {
                def RESULT = sh returnStatus: true, script: 'vendor/bin/phpmd ./ xml phpmd.xml --reportfile ./build/logs/phpmd.log.xml --exclude ./vendor, autoload.php, ./var, ./node_modules, ./templates, ./public'
                pmd canRunOnFailed: true, defaultEncoding: 'UTF-8', pattern: 'build/logs/phpmd.log.xml'
                
                if ( RESULT != 0 ) 
                {
                    currentBuild.result = 'FAILURE'
                    error 'Des erreurs ont été détectées pour PHPMD : ${RESULT}'
                }
            }
        }
    }
}
