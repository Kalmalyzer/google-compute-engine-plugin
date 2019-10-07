/*
 * Copyright 2019 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            label 'maven-pod'
            yamlFile 'jenkins/maven-pod.yaml'
        }
    }

    environment {
        GOOGLE_PROJECT_ID = "${GCE_IT_PROJECT_ID}"
        GOOGLE_REGION = "${GCE_IT_REGION}"
        GOOGLE_ZONE = "${GCE_IT_ZONE}"
        GOOGLE_SA_NAME = "${GCE_IT_SA}"
        FAILED_ARTIFACTS_BUCKET = "${GCE_IT_BUCKET}"
        FAILED_ARTIFACTS = "failed-${BRANCH_NAME}-${BUILD_ID}.tar.gz"
    }

    stages {
        stage("Build and test") {
            steps {
                container('maven') {
                    withCredentials([[$class: 'StringBinding', credentialsId: env.GCE_IT_CRED_ID, variable: 'GOOGLE_CREDENTIALS']]) {
                        catchError {
                            // build
                            // sh "mvn clean package -ntp"

                            // run tests
                            sh "mvn verify -ntp -Dit.test=ConfigAsCodeTestIT"
                        }

                        sh "jenkins/saveAndCompress.sh"
                        step([$class: 'ClassicUploadStep', credentialsId: env.GCE_BUCKET_CRED_ID, bucket: "gs://${FAILED_ARTIFACTS_BUCKET}", pattern: env.FAILED_ARTIFACTS])
                    }
                }
            }
        }
    }
}
