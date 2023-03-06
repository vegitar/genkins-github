pipeline {
    // 스테이지 별로 다른 거
    agent any

    triggers {
        pollSCM('*/3 * * * *')  //3분 주기
    }

    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
    }
    //AWS용 접속 크리덴셜 -사용자 보안자격증명 액세스 키 만들기

    stages {        //* 파이프 라인의 구성 - 큰 덩어리
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            
            steps {
                echo 'Clonning Repository'

                git url: 'https://github.com/vegitar/genkins-github.git',
                    branch: 'master',
                    credentialsId: 'Jenkins-Github' //defined in credentials area ?
                                            // GitHub Servers 의 Credentials?
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Successfully Cloned Repository'
                }

                always { 
                  echo "i tried..."
                }

                cleanup {
                  echo "after all other post condition"
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://vegitartest
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  mail  to: 'itsecplanet@gmail.com',
                        subject: "Deploy Frontend Success",
                        body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  mail  to: 'itsecplanet@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh """
                docker build . -t server --build-arg env=${PROD}
                """
            }
          }

          post {
            failure {
              error 'This pipeline stops here...' 
              //서버 빌드하다가 에러나면 여기서 종료시켜버려라
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh '''
                
                docker run -p 80:80 -d server 
                '''
                //위에서 서버라는 이름으로 태그를 붙인 걸 그대로 받아서 사용
                //docker rm -f $(docker ps -aq) 원래 도커에서 돌고 있는 것이 있다면 괄호안 명령러로 
                //일단 꺼버리고 개체를 지워줘라
                //여기 실습에서는 아무것도 없기 때문에  dockeer run -p 앞에 위치한 이 라인을 제거
                //두번 째 실습부터는 넣어주면 됨
            }
          }

          post {
            success {
              mail  to: 'frontalnh@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
}
