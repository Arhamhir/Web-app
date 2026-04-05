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
    JWT_SECRET_KEY = 'jenkins-part2-secret-change-me'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Backend Build Check (Containerized)') {
      steps {
        script {
          docker.image('python:3.12-slim').inside('-v $WORKSPACE/backend:/app -w /app') {
            sh 'pip install --no-cache-dir -r requirements.txt'
            sh 'python -m compileall app'
          }
        }
      }
    }

    stage('Frontend Build Check (Containerized)') {
      steps {
        script {
          docker.image('node:20-alpine').inside('-v $WORKSPACE/frontend:/app -w /app') {
            sh 'npm install'
            sh 'VITE_API_URL=http://${PUBLIC_IP}:8201 npm run build'
          }
        }
      }
    }

    stage('Deploy Part2 Stack') {
      steps {
        sh 'docker compose -f $COMPOSE_FILE pull'
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
