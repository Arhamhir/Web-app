pipeline {
  agent any

  triggers {
    githubPush()
  }

  options {
    timestamps()
  }

  parameters {
    string(name: 'PUBLIC_IP', defaultValue: '35.170.34.8', description: 'EC2 Elastic IP used by frontend API URL')
  }

  environment {
    COMPOSE_FILE = 'docker-compose.part2.yml'
    POSTGRES_PASSWORD = 'taskflow123'
    JWT_SECRET_KEY = 'secret'
    DOCKERHUB_USERNAME = 'arhamheer'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Backend Image') {
      steps {
        sh 'docker build -t ${DOCKERHUB_USERNAME}/taskflow-backend:part1 ./backend'
      }
    }

    stage('Build Frontend Image') {
      steps {
        sh 'docker build --build-arg VITE_API_URL=http://${PUBLIC_IP}:8201 -t ${DOCKERHUB_USERNAME}/taskflow-frontend:part2 ./frontend'
      }
    }

    stage('Push Part2 Images') {
      steps {
        sh 'docker push ${DOCKERHUB_USERNAME}/taskflow-backend:part1'
        sh 'docker push ${DOCKERHUB_USERNAME}/taskflow-frontend:part2'
      }
    }

    stage('Deploy Part2 Stack') {
      steps {
        sh 'docker compose -f $COMPOSE_FILE down --remove-orphans || true'
        sh 'PUBLIC_IP=${PUBLIC_IP} POSTGRES_PASSWORD=${POSTGRES_PASSWORD} JWT_SECRET_KEY=${JWT_SECRET_KEY} docker compose -f $COMPOSE_FILE up -d --force-recreate'
      }
    }

    stage('Health Check') {
      steps {
        sh 'curl -fsS http://localhost:8201/'
        sh 'curl -fsSI http://localhost:8200/'
      }
    }
  }
}
