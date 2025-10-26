pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'portfolio-website'
        DOCKER_TAG = "${BUILD_NUMBER}"
        REGISTRY_URL = 'your-registry.com'
        STAGING_URL = 'https://staging.yourportfolio.com'
        PRODUCTION_URL = 'https://yourportfolio.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm clean-install'
            }
        }
        
        stage('Parallel Quality Checks') {
            parallel {
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
                
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:coverage'
                        publishTestResults testResultsPattern: 'coverage/junit.xml'
                        publishCoverageResults([
                            [
                                reportDir: 'coverage',
                                reportFiles: 'clover.xml',
                                reportName: 'Coverage Report'
                            ]
                        ])
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        sh 'npm audit --audit-level moderate'
                    }
                }
                
                stage('Build Assets') {
                    steps {
                        sh 'npm run build'
                        archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
                    }
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Development Image') {
                    steps {
                        script {
                            docker.build("${DOCKER_IMAGE}:dev-${DOCKER_TAG}", "--target development .")
                        }
                    }
                }
                
                stage('Production Image') {
                    steps {
                        script {
                            dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", "--target production .")
                            docker.build("${DOCKER_IMAGE}:latest", "--target production .")
                        }
                    }
                }
            }
        }
        
        stage('Docker Security Scan') {
            steps {
                script {
                    sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
        
        stage('Integration Tests') {
            steps {
                script {
                    sh '''
                        docker-compose -f docker-compose.test.yml up --build -d
                        sleep 15
                        
                        # Test if the application is responding
                        curl -f http://localhost:3000 || exit 1
                        
                        # Run additional integration tests
                        docker-compose -f docker-compose.test.yml exec -T portfolio-dev npm run test
                        
                        docker-compose -f docker-compose.test.yml down
                    '''
                }
            }
        }
        
        stage('Performance Tests') {
            steps {
                script {
                    sh '''
                        # Start the application
                        docker run -d --name portfolio-perf -p 8081:80 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        sleep 10
                        
                        # Run Lighthouse CI for performance testing
                        npx lighthouse-ci autorun --upload.target=temporary-public-storage || true
                        
                        # Cleanup
                        docker stop portfolio-perf
                        docker rm portfolio-perf
                    '''
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                script {
                    // Tag and push staging image
                    sh '''
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${REGISTRY_URL}/${DOCKER_IMAGE}:staging-${DOCKER_TAG}
                        docker push ${REGISTRY_URL}/${DOCKER_IMAGE}:staging-${DOCKER_TAG}
                    '''
                    
                    // Deploy to staging environment
                    sh '''
                        # Update staging deployment
                        kubectl set image deployment/portfolio-staging \
                            portfolio=${REGISTRY_URL}/${DOCKER_IMAGE}:staging-${DOCKER_TAG} \
                            --namespace=staging
                        kubectl rollout status deployment/portfolio-staging --namespace=staging
                        
                        # Wait for deployment to be ready
                        kubectl wait --for=condition=available --timeout=300s deployment/portfolio-staging --namespace=staging
                    '''
                    
                    // Run smoke tests against staging
                    sh '''
                        sleep 30
                        curl -f ${STAGING_URL} || exit 1
                        curl -f ${STAGING_URL}/health || exit 1
                    '''
                }
            }
        }
        
        stage('Staging Tests') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                script {
                    sh '''
                        # Run end-to-end tests against staging
                        npm run test:e2e -- --baseUrl=${STAGING_URL} || true
                        
                        # Run accessibility tests
                        npx pa11y ${STAGING_URL} || true
                        
                        # Run SEO tests
                        npx lighthouse ${STAGING_URL} --only-categories=seo --chrome-flags="--headless" || true
                    '''
                }
            }
        }
        
        stage('Production Deployment Approval') {
            when {
                branch 'main'
            }
            steps {
                script {
                    env.DEPLOY_TO_PROD = input message: 'Deploy to Production?',
                        ok: 'Deploy',
                        parameters: [
                            choice(name: 'DEPLOY_TO_PROD', choices: 'no\nyes', description: 'Choose yes to deploy to production'),
                            text(name: 'DEPLOYMENT_NOTES', defaultValue: '', description: 'Deployment notes (optional)')
                        ]
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                allOf {
                    branch 'main'
                    environment name: 'DEPLOY_TO_PROD', value: 'yes'
                }
            }
            steps {
                script {
                    // Create production release
                    sh '''
                        # Tag and push production image
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${REGISTRY_URL}/${DOCKER_IMAGE}:prod-${DOCKER_TAG}
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${REGISTRY_URL}/${DOCKER_IMAGE}:latest
                        docker push ${REGISTRY_URL}/${DOCKER_IMAGE}:prod-${DOCKER_TAG}
                        docker push ${REGISTRY_URL}/${DOCKER_IMAGE}:latest
                    '''
                    
                    // Blue-Green deployment to production
                    sh '''
                        # Deploy to production with zero downtime
                        kubectl set image deployment/portfolio-production \
                            portfolio=${REGISTRY_URL}/${DOCKER_IMAGE}:prod-${DOCKER_TAG} \
                            --namespace=production
                        kubectl rollout status deployment/portfolio-production --namespace=production
                        
                        # Wait for deployment to be ready
                        kubectl wait --for=condition=available --timeout=600s deployment/portfolio-production --namespace=production
                    '''
                    
                    // Post-deployment verification
                    sh '''
                        sleep 60
                        curl -f ${PRODUCTION_URL} || exit 1
                        curl -f ${PRODUCTION_URL}/health || exit 1
                        
                        # Run critical path tests
                        npm run test:critical -- --baseUrl=${PRODUCTION_URL} || exit 1
                    '''
                }
            }
        }
        
        stage('Post-Deployment') {
            when {
                allOf {
                    branch 'main'
                    environment name: 'DEPLOY_TO_PROD', value: 'yes'
                }
            }
            steps {
                script {
                    // Update monitoring and alerting
                    sh '''
                        # Update deployment tracking
                        curl -X POST "${MONITORING_WEBHOOK}\" \
                            -H "Content-Type: application/json\" \
                            -d "{
                                \\"deployment\\": \\"${DOCKER_TAG}\\",
                                \\"environment\\": \\"production\\",
                                \\"status\\": \\"success\\",
                                \\"url\\": \\"${PRODUCTION_URL}\\",
                                \\"commit\\": \\"${GIT_COMMIT_SHORT}\\"
                            }" || true
                    '''
                    
                    // Create GitHub release
                    sh '''
                        gh release create v${DOCKER_TAG} \
                            --title "Portfolio Website v${DOCKER_TAG}\" \
                            --notes "Automated release from Jenkins pipeline\" \
                            --target main || true
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup
            sh '''
                docker system prune -f
                docker volume prune -f
            '''
            cleanWs()
        }
        
        success {
            script {
                def message = "✅ Portfolio deployment succeeded!"
                if (env.DEPLOY_TO_PROD == 'yes') {
                    message += "\n🚀 Live at: ${PRODUCTION_URL}"
                } else if (env.BRANCH_NAME in ['develop', 'main']) {
                    message += "\n🧪 Staging: ${STAGING_URL}"
                }
                
                slackSend(
                    channel: '#deployments',
                    color: 'good',
                    message: "${message}\nBuild: ${env.BUILD_NUMBER} | Commit: ${env.GIT_COMMIT_SHORT}"
                )
            }
        }
        
        failure {
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: "❌ Portfolio deployment failed!\nBuild: ${env.BUILD_NUMBER} | Branch: ${env.BRANCH_NAME}\n<${env.BUILD_URL}|View Build>"
            )
        }
        
        unstable {
            slackSend(
                channel: '#deployments',
                color: 'warning',
                message: "⚠️ Portfolio deployment unstable!\nBuild: ${env.BUILD_NUMBER} | Branch: ${env.BRANCH_NAME}\n<${env.BUILD_URL}|View Build>"
            )
        }
    }
}