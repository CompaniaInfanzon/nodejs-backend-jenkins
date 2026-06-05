pipeline {
    agent {
        docker {
            image 'devops-agent:latest'
            // Este argumento es vital para que puedas usar comandos de docker dentro del agente
            args '-v /var/run/docker.sock:/var/run/docker.sock' 
        }
    }
    
    environment {
        // Variables vinculadas a las credenciales creadas en el Lab 7
        AZURE_CLIENT_ID = credentials('azure-clientId')
        AZURE_CLIENT_SECRET = credentials('azure-clientSecret')
        AZURE_TENANT_ID = credentials('azure-tenantId')
        AZURE_SUBSCRIPTION_ID = credentials('azure-subscriptionId')
    }

    stages {
        stage('[CI] Instalar dependencias de app') {
            steps {
                sh 'npm install'
            }
        }
        stage('[CI] Ejecutar pruebas unitarias') {
            steps {
                sh 'npm run test'
            }
        }
        stage('[CI] Ejecutar pruebas de integracion') {
            steps {
                sh 'npm run test:integration'
            }
        }
        stage('[CI] Azure Login') {
            steps {
                sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID'
                sh 'az account set --subscription $AZURE_SUBSCRIPTION_ID'
            }
        }
        stage('[CI] AKS Credentials') {
            steps {
                // Conectando al clúster que acabas de crear con tus datos reales
                sh 'az aks get-credentials --resource-group rg-cicd-terraform-app-infanzon --name aks-infanzon'
            }
        }
        stage('[CI] Generar ID corto del commit') {
            steps {
                script {
                    env.SHORT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Commit ID: ${env.SHORT_SHA}"
                }
            }
        }
        stage('[CI] Build and Push Docker Image') {
            steps {
                // Usando tu Azure Container Registry real
                sh "docker build -t acrinfanzon.azurecr.io/nodejs-backend-jenkins:latest ."
                sh "docker push acrinfanzon.azurecr.io/nodejs-backend-jenkins:latest"
            }
        }
        
        // ---------------- DESPLIEGUE DEV ----------------
        stage('[CD-DEV] Deploy a AKS') {
            steps {
                sh 'kubectl apply -f k8s.yml -n dev'
            }
        }
        stage('[CD-DEV] Imprimir IP del servicio') {
            steps {
                sh 'kubectl get svc -n dev'
            }
        }

        // ---------------- DESPLIEGUE QA ----------------
        stage('Aprobacion QA') {
            steps {
                input message: '¿Aprobar pase a entorno de QA?'
            }
        }
        stage('[CD-QA] Deploy a AKS') {
            steps {
                sh 'kubectl apply -f k8s.yml -n qa'
            }
        }
        stage('[CD-QA] Imprimir IP del servicio') {
            steps {
                sh 'kubectl get svc -n qa'
            }
        }

        // ---------------- DESPLIEGUE PRD ----------------
        stage('Aprobacion PRD') {
            steps {
                input message: '¿Aprobar pase a entorno de PRODUCCION?'
            }
        }
        stage('[CD-PRD] Deploy a AKS') {
            steps {
                sh 'kubectl apply -f k8s.yml -n prd'
            }
        }
        stage('[CD-PRD] Imprimir IP del servicio') {
            steps {
                sh 'kubectl get svc -n prd'
            }
        }
    }
}