pipeline {
  agent {
    kubernetes {
      cloud 'kubernetes'
      label 'jenkins-agent'
      defaultContainer 'docker'
    }
  }
  environment { 
        app = 'book-api'
  }
  stages {
    stage('Create Image') {
        when {
            expression {
                return env.GIT_BRANCH ==~ '.*/master'
            }
        } 
      steps {
        container('docker') {
        sh '''
            set -e
            env
            ls -arlt
            git config --global --add safe.directory $PWD
            commitid=$(git log -1 --format=%h)
            echo $commitid
            docker build -t $registryUrl/library/$app:latest -t $registryUrl/library/$app:$commitid -f src/Dockerfile .
            docker images 
            echo "**************"
            echo "Docker login"
            echo $registryPassword | docker login -u $registryUser $registryUrl --password-stdin
            echo "Docker pushing images"
            docker push $registryUrl/library/$app:latest
            
        '''
        }
      }
    }
    stage('Deploy Development') {
        when {
            expression {
                return env.GIT_BRANCH ==~ '.*/master'
            }
        } 
      steps {
        container('k8s') {
        sh '''
        set -e
        git config --global --add safe.directory $PWD
        commitId=$(git log -1 --format=%h)
        echo $commitId
        echo "$registryUrl/library/$app:$commitId"
        kubectl set image deployment/book-api main=$registryUrl/library/$app:$commitId -n development
        '''
        }
      }
    }
    stage('Deploy Production') {
    when {
        expression {
            return env.GIT_BRANCH ==~ 'refs/tags/v1.*..*..*'
        }
    } 
      steps {
        container('docker') {
        sh '''
        set -e
        git config --global --add safe.directory $PWD
        commitId=$(git log -1 --format=%h)
        echo $commitId
        echo "$registryUrl/library/$app:$commitId"
        echo "Docker login"
        echo $registryPassword | docker login -u $registryUser $registryUrl --password-stdin
        docker pull $registryUrl/library/$app:$commitId
        docker tag $registryUrl/library/$app:$commitId $registryUrl/$app:latest-release
        docker push $registryUrl/library/$app:latest-release
        '''
        }
        container('k8s') {
        sh '''
        set -e
        git config --global --add safe.directory $PWD
        commitId=$(git log -1 --format=%h)
        echo $commitId
        echo "$registryUrl/library/$app:$commitId"
        kubectl set image deployment/$app main=$registryUrl/library/$app:$commitId -n production
        '''
        }
      }
    }
  }
}