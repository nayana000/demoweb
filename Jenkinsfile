pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET  = 'test'
        VERSION    = "${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh '''
                    echo "Building version ${VERSION}"

                    # Check that index.html exists
                    test -f web.html

                    # Create deployment directory
                    rm -rf deployment
                    mkdir -p deployment

                    # Copy the original HTML
                    cp web.html deployment/index.html

                    # Replace placeholder with Jenkins build number
                    sed -i "s/VERSION_PLACEHOLDER/${VERSION}/g" \
                        deployment/web.html

                    echo "Generated website:"
                    cat deployment/web.html
                '''
            }
        }

        stage('Deploy Version') {
            steps {
                withAWS(
                    credentials: 'awscredentials',
                    region: "${AWS_REGION}"
                ) {
                    sh '''

                        echo "Deploying version v${VERSION}"

                        aws s3 cp \
                            deployment/web.html \
                            "s3://${S3_BUCKET}/versions/v${VERSION}/web.html"
                    '''
                }
            }
        }

        stage('Deploy Latest') {
            steps {
                withAWS(
                    credentials: 'awscredentials',
                    region: "${AWS_REGION}"
                ) {
                    sh '''

                        echo "Deploying v${VERSION} as latest"

                        aws s3 cp \
                            deployment/web.html \
                            "s3://${S3_BUCKET}/web.html"

                        echo "${VERSION}" > deployment/latest-version.txt

                        aws s3 cp \
                            deployment/latest-version.txt \
                            "s3://${S3_BUCKET}/latest-version.txt"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo """
            =====================================
            DEPLOYMENT SUCCESSFUL
            =====================================

            Version: v${VERSION}

            Versioned:
            s3://${S3_BUCKET}/versions/v${VERSION}/web.html

            Latest:
            s3://${S3_BUCKET}/web.html

            =====================================
            """
        }

        failure {
            echo "Deployment failed for version v${VERSION}"
        }

        always {
            sh '''
                rm -rf deployment || true
            '''
        }
    }
}
