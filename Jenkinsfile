pipeline {
    agent any
    environment {
        CONTAINER_ID = ''
        SUM_PY_PATH = '/home/la3fiif/Documents/FinalProjectDevOps/sum.py'
        DIR_PATH = '/home/la3fiif/Documents/FinalProjectDevOps'
        TEST_FILE_PATH = '/home/la3fiif/Documents/FinalProjectDevOps/test_variables.txt'
        
        IMAGE_NAME = "sum-app"
        CONTAINER_NAME = "sum-container"
        DOCKERHUB_IMAGE = "laa3fiif/sum-app:latest"
    }

stages {
stage('Build') {
    steps {
        dir("${DIR_PATH}") {
            sh "docker build -t %IMAGE_NAME% ."
        }
    }
}

stage('Run') {
    steps {
        script {
            sh "docker rm -f %CONTAINER_NAME% || exit 0"

            def output = sh(
              script: "docker run -d %CONTAINER_NAME% %IMAGE_NAME%",
              returnStdout: true
            )
            CONTAINER_ID = output.split("\\r?\\n")[-1].trim()
            echo "Container started: name=%CONTAINER_NAME%, id=${CONTAINER_ID}"
        }
    }
}

stage('Test') {
    steps {
        script {
            def lines = readFile("${TEST_FILE_PATH}").trim().split("\\r?\\n")

            for (def line : lines) {
                line = line.trim()
                if (line == "") continue

                def v = line.split("\\s+")
                def arg1 = v[0]
                def arg2 = v[1]
                def expected = v[2].toFloat()

                def output = sh(
                    script: "docker exec %CONTAINER_NAME% python /app/sum.py ${arg1} ${arg2}",
                    returnStdout: true
                )
          
                def outlines = output.split("\\r?\\n")
                def result = outlines[-1].trim()

                def ok = sh(
                    script: """docker exec %CONTAINER_NAME% python -c "from decimal import Decimal; \
a=Decimal('${arg1}'); b=Decimal('${arg2}'); e=Decimal('${expected}'); print(a+b==e)" """,
                    returnStdout: true
                ).trim()

                if (ok.endsWith("True")) {
                    echo "OK ${arg1} + ${arg2} = ${result}"
                } else {
                    error "NOPE. Expected ${expected}, got ${result}"
                } 
            }
        }
    }
}

stage('Deploy') {
    steps {
        withCredentials([usernamePassword(
        credentialsId: 'dockerhub-creds',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
        )]) {
        
        script {
        sh """
        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
        docker tag %IMAGE_NAME% %DOCKERHUB_IMAGE%
        docker push %DOCKERHUB_IMAGE%
        """
    }
}
}
}

post {
    always {
        script {
            sh "docker rm -f %CONTAINER_NAME% || exit 0"
        }
    }
}


}
}
