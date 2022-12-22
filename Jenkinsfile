pipeline {
  agent any
  parameters {
    choice (name: 'chooseNode', choices: ['Green', 'Blue'], description: 'Choose which Environment to Deploy: ')
  }
  environment {
    listenerARN = 'arn:aws:elasticloadbalancing:ap-south-1:007591908515:listener/app/ALBalancer/207ca4ea071aa189/c2cd03929b2cc07b'
    blueARN = 'arn:aws:elasticloadbalancing:ap-south-1:007591908515:targetgroup/bluetarget/5cd20def2f63a298'
    greenARN = 'arn:aws:elasticloadbalancing:ap-south-1:007591908515:targetgroup/greentarget/2066a4dd1612223d'
  }
  stages {
    stage('Deployment Started') {
      parallel {
        stage('Green') {
          when {
            expression {
              params.chooseNode == 'Green'
            }
          }
          stages {
            // stage('Offloading Green') {
            //   steps {
            //     sh """aws elbv2 modify-listener – listener-arn ${listenerARN} – default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
            //   }
            // }
            stage('Deploying to Green') {
              steps {
                sh '''scp -r index.html ec2-user@15.207.115.110:/usr/share/nginx/html/green/
                ssh -t ec2-user@15.207.115.110 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate and Add Green for testing') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null – silent – head – write-out '%{http_code}' http://15.207.115.110/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://15.207.115.110/
                    aws elbv2 modify-listener – listener-arn ${listenerARN} – default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
                    echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://15.207.115.110/
                exit 2
                fi
                """
              }
            }
          }
        }
        stage('Blue') {
          when {
            expression {
              params.chooseNode == 'Blue'
            }
          }
          stages {
            stage('Offloading Blue') {
              steps {
                sh """aws elbv2 modify-listener – listener-arn ${listenerARN} – default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 0 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
              }
            }
            stage('Deploying to Blue') {
              steps {
                sh '''scp -r index.html ec2-user@3.111.149.194:/usr/share/nginx/html/blue/
                ssh -t ec2-user@3.111.149.194 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate Blue and added to TG') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null – silent – head – write-out '%{http_code}' http://3.111.149.194/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://3.111.149.194/
                    aws elbv2 modify-listener – listener-arn ${listenerARN} – default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
                    echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://3.111.149.194/
                exit 2
                fi
                """
              }
            }
          }
        }
      }
    }
  }
}
