pipeline
 {
  agent any


  /*
  parameters
   {
    // string(name: 'WITH_CONFIG', defaultValue: 'localhost', description: 'Build with configuration for a specific host.')
    // booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: 'Build as debug or production version.')
    // text(name: 'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '')
    // choice(name: 'CHOICES', choices: ['one', 'two', 'three'], description: '')
    // file(name: 'FILE', description: 'Some file to upload')
    // password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A secret password')
   }
  */

  tools
   {
    maven 'Maven3'
    jdk 'JDK11'
    git 'GIT'
   }


  options
   {
    buildDiscarder(logRotator(numToKeepStr: '4'))
    skipStagesAfterUnstable()
    disableConcurrentBuilds()
    // timestamps()
    // timeout(time: 1, unit: 'HOURS')
    // parallelsAlwaysFailFast()
   }


  triggers
   {
    // MINUTE HOUR DOM MONTH DOW
    pollSCM('H 6-18/4 * * 1-5')
   }


  environment
   {
    DISABLE_AUTH = 'true'
    expectedRemoteUrl = "https://github.com/PowerStat/ICal2Xml.git"
   }


  stages
   {
    stage('Clean')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode clean'
           }
          else
           {
            bat 'mvn --batch-mode clean'
           }
         }
       }
     }

    stage('Build')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode compile'
           }
          else
           {
            bat 'mvn --batch-mode compile'
           }
         }
       }
     }

    stage('UnitTests')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode resources:testResources compiler:testCompile surefire:test'
           }
          else
           {
            bat 'mvn --batch-mode resources:testResources compiler:testCompile surefire:test'
            //  -Dmaven.test.failure.ignore=true
           }
         }
       }
      post
       {
        always
         {
          junit testResults: 'target/surefire-reports/*.xml'
         }
       }
     }

    stage('MutationTests')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sht 'mvn --batch-mode org.pitest:pitest-maven:mutationCoverage'
           }
          else
           {
            bat 'mvn --batch-mode org.pitest:pitest-maven:mutationCoverage'
           }
         }
       }
     }

    stage('Sanity check')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode checkstyle:checkstyle pmd:pmd pmd:cpd com.github.spotbugs:spotbugs-maven-plugin:spotbugs'
           }
          else
           {
            bat 'mvn --batch-mode checkstyle:checkstyle pmd:pmd pmd:cpd com.github.spotbugs:spotbugs-maven-plugin:spotbugs'
           }
         }
        // scanForIssues tools: [java(), javaDoc(), eclipse(), cssLint(), groovyScript(), jsLint()]
        // sonarQube
        // yuiCompressor
        // dependencyCheckAnalyzer
        // dependencyCheckPublisher
        // dependencyTrackPublisher
        // Arachni Scanner Plugin
        // Black Duck Detect
        /*
        codesonar
        withSonarQubeEnv('My SonarQube Server')
         {
          bat 'mvn --batch-mode sonar:sonar'
         }
        waitForQualityGate
        */
       }
      /*
      post
       {
        always
         {
          // recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc(), checkStyle(), spotBugs(), cpd(pattern: '** /target/cpd.xml'), pmdParser(pattern: '** /target/pmd.xml')]
         }
       }
      */
     }

    stage('Packaging')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode jar:jar'
           }
          else
           {
            bat 'mvn --batch-mode jar:jar'
           }
         }
       }
     }

    stage('Install local')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode jar:jar install:install'
           }
          else
           {
            bat 'mvn --batch-mode jar:jar install:install' // maven-jar-plugin falseCreation default is false, so no doubled jar construction here, but required for maven-install-plugin internal data
           }
         }
       }
     }

    stage('Integration tests')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode failsafe:integration-test failsafe:verify'
           }
          else
           {
            bat 'mvn --batch-mode failsafe:integration-test failsafe:verify'
           }
         }
       }
     }

    stage('Documentation')
     {
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode -Dweb.server=www.powerstat.de site'
           }
          else
           {
            bat 'mvn --batch-mode -Dweb.server=www.powerstat.de site'
            // Change history
           }
         }
       }
      post
       {
        always
         {
          publishHTML(target: [reportName: 'Site', reportDir: 'target/site', reportFiles: 'index.html', keepAll: false])
          // publishHTML(target: [reportName: 'Manual', reportDir: 'target/generated-docs', reportFiles: 'TemplateEngine.html', keepAll: false])
         }
       }
     }

    // Security tests
    // Fault tolerance tests
    // Performance tests cucumber

    /*
    stage('Release and deploy')
     {
      when
       {
        // settings.xml contain servers,profiles: ossrh, powerstat
        expression
         {
          def remoteUrl = isUnix() ? sh(script: "git config remote.origin.url", returnStdout: true)?.trim() : bat(script: "git config remote.origin.url", returnStdout: true)?.trim()
          return 'https://github.com/PowerStat/TemplateEngine.git' == remoteUrl
         }
       }
      steps
       {
        script
         {
          if (isUnix())
           {
            sh 'mvn --batch-mode release:clean'
            sh 'mvn --batch-mode release:prepare'
            sh 'mvn --batch-mode release:perform'
            git push -–tags
            git push origin master
           }
          else
           {
            bat 'mvn --batch-mode release:clean'
            bat 'mvn --batch-mode release:prepare'
            bat 'mvn --batch-mode release:perform'
            git push -–tags
            git push origin master
           }
         }
       }
     }
    */

   }

 }
