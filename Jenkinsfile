
pipeline {

    agent any
    
    stages {
        stage('GIT') 
        {
            steps{
             checkout([$class: 'GitSCM', branches: [[name: '*/main']],
              userRemoteConfigs: [[url: 'https://github.com/ramtejb/CppUnit_Wrk.git']]])   
            }
          
        }
        stage('Build') {
            steps {
                sh '''
                    g++ -o testBasicMath CBasicMath.cpp TestBasicMath.cpp -lcppunit
                '''
            }

            post {
                success {
                   archiveArtifacts artifacts: 'testBasicMath', fingerprint: true
                }
            }
        }
        
        stage('Test') {
            steps {
                sh './testBasicMath'
                
            }
			
			 post {
                always{
                xunit (
                tools: [ CppUnit(pattern: 'cppTestBasicMathResults.xml') ])
               }
            }
        }
		stage('StaticAnalysis'){
			steps {
				sh '''
				cppcheck --enable=all --inconclusive --xml --xml-version=2 "${WORKSPACE}" 2> cppcheck.xml
				'''
			}
			
			post{
				always{
				publishCppcheck pattern:'cppcheck.xml'
				}
			}
			
			
		}
		
         stage('Push') {
            steps {
                sh '''echo "Publishing on Github..."
                echo "$PWD"
                token="8d8f694d19dd3f77d2db696fdf5019c25664e501"
                release=$(curl -XPOST -H "Authorization:token $token" --data "{\\"tag_name\\": \\"${BUILD_NUMBER}\\", \\"target_commitish\\": \\"main\\", \\"name\\": \\"JenkinsRelease\\", \\"body\\": \\"JenkinsRelease v${BUILD_NUMBER}\\", \\"draft\\": false, \\"prerelease\\": false}" https://api.github.com/repos/ramtejb/CppUnit_Wrk/releases)
                # Extract the id of the release from the creation response
                id=$(echo "$release" | sed -n -e \'s/"id":\\ \\([0-9]\\+\\),/\\1/p\' | head -n 1 | sed \'s/[[:blank:]]//g\')
                # Upload the artifact
                curl -XPOST -H "Authorization:token $token" -H "Content-Type:application/octet-stream" --data-binary testBasicMath_BIN.zip https://uploads.github.com/repos/ramtejb/CppUnit_Wrk/releases/$id/assets?name=testBasicMath_BIN.zip'''
             }
        }

        stage('Deploy') {
            steps {
                echo "Deploy "
            }
        }
    }
}