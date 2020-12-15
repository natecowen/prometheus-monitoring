/*************************************************************************
* Prometheus Monitoring
*
* JENKINS SETUP
*	 1) Add project type "Multibranch Pipeline"
*    2) Add source "Git":
*      • Project Repository "http://github.com/natecowen/prometheus-monitoring.git"
*      • Credentials: JenkinsBot/*****'
*       • Behaviors > Filter by name (with regular expression): "^(?!archive|feature|legacy).*$" (don't include quotes)

**************************************************************************/

pipeline {
	agent { node { label "linux" } }
	options { timestamps() }
	environment {
        K8_CONTEXT_DEVELOPMENT="dev-things@k8-green"
        K8_CONTEXT_BLUE_DEVELOPMENT="dev-things@k8-blue"

		REGISTRY="registry.something.com"
        GRAFANA_URL="https://developer.something.com:30004/api/org/preferences"
	}

    stages {
		stage ("Pre-flight") {
            agent { node { label "linux" } }
            steps {
                    sh "sudo netplan apply"
					sh "docker version"
                    sh "command -v kubectl"

                    // Verify the K8 contexts are valid and reachable via kubectl
						sh "kubectl config use-context '${K8_CONTEXT_DEVELOPMENT}'"
            }
        }

        stage ("Build Docker Images") {
            parallel {
                stage("Prometheus Monitor") {
                    steps {
                        dir("${env.WORKSPACE}/docker/prometheus-docker/") {
                            sh "docker build --no-cache -t ${REGISTRY}/prom-monitor:${env.GIT_COMMIT} -t ${REGISTRY}/prom-monitor:latest  ."
                        }
                    }
                }
                stage("Prometheus Blackbox") {
                    steps {
                        dir("${env.WORKSPACE}/docker/blackbox-docker/") {
                            sh "docker build --no-cache -t ${REGISTRY}/prom-blackbox:${env.GIT_COMMIT} -t ${REGISTRY}/prom-blackbox:latest  ."
                        }
                    }
                }

                stage("Prometheus Alert Manger") {
                    steps {
                        dir("${env.WORKSPACE}/docker/alert-manager/") {
                            sh "docker build --no-cache -t ${REGISTRY}/prom-altermanager:${env.GIT_COMMIT} -t ${REGISTRY}/prom-altermanager:latest  ."
                        }
                    }
                }

                stage("Prom2Teams") {
                    steps {
                        dir("${env.WORKSPACE}/docker/prom2teams-docker/") {
                            sh "docker build --no-cache -t ${REGISTRY}/prom-toteams:${env.GIT_COMMIT} -t ${REGISTRY}/prom-toteams:latest  ."
                        }
                    }
                }

                stage("Grafana") {
                    steps {
                        dir("${env.WORKSPACE}/docker/grafana-docker/") {
                            sh "docker build --no-cache -t ${REGISTRY}/grafana:${env.GIT_COMMIT} -t ${REGISTRY}/grafana:latest  ."
                        }
                    }
                }

                stage("Mongo Monitor") {
                    steps {
                        dir("${env.WORKSPACE}/docker/mongo/") {
                            sh "docker build -t ${REGISTRY}/mongoexporter:${env.GIT_COMMIT} -t ${REGISTRY}/mongoexporter:latest https://github.com/dcu/mongodb_exporter.git "
                        }
                    }
                }
            }

           
        }


        stage ("Docker Registry Push") {
            agent { node { label "linux" } }
            steps {
                dir("${env.WORKSPACE}/docker/") {
                    sh "sudo netplan apply"


                    // SECURE PRIVATE DOCKER REGISTRY
                    withCredentials([usernamePassword(credentialsId: 'credentials-docker-registry', passwordVariable: 'DockerPass', usernameVariable: 'DockerUser')]) {

                        sh "echo $DockerPass | docker login --username \"$DockerUser\" --password-stdin \"${REGISTRY}\" "
                        sh "docker push ${REGISTRY}/prom-monitor:${env.GIT_COMMIT} && docker push ${REGISTRY}/prom-monitor:latest"
                        sh "docker push ${REGISTRY}/prom-blackbox:${env.GIT_COMMIT} && docker push ${REGISTRY}/prom-blackbox:latest"
                        sh "docker push ${REGISTRY}/prom-altermanager:${env.GIT_COMMIT} && docker push ${REGISTRY}/prom-altermanager:latest"
                        sh "docker push ${REGISTRY}/prom-toteams:${env.GIT_COMMIT} && docker push ${REGISTRY}/prom-toteams:latest"
                        sh "docker push ${REGISTRY}/grafana:${env.GIT_COMMIT} && docker push ${REGISTRY}/grafana:latest"
                        sh "docker push ${REGISTRY}/mongoexporter:${env.GIT_COMMIT} && docker push ${REGISTRY}/mongoexporter:latest"
                    }

                }
            }
        }
        stage ("Deploy to Kubernetes") {
            parallel {
                stage ("Prometheus Monitor") {
                    steps {

                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/prometheus-deployment prometheus=${REGISTRY}/prom-monitor:${env.GIT_COMMIT} --context='${K8_CONTEXT_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/prometheus-deployment --context='${K8_CONTEXT_DEVELOPMENT}';";
                        sh "kubectl get deployment/prometheus-deployment -o wide --context='${K8_CONTEXT_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"

                         // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/prometheus-deployment prometheus=${REGISTRY}/prom-monitor:${env.GIT_COMMIT} --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/prometheus-deployment --context='${K8_CONTEXT_BLUE_DEVELOPMENT}';";
                        sh "kubectl get deployment/prometheus-deployment -o wide --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"
                    }
                }

                stage ("Prometheus Blackbox") {
                    steps {

                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/blackbox-deployment prometheus-blackbox=${REGISTRY}/prom-blackbox:${env.GIT_COMMIT} --context='${K8_CONTEXT_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/blackbox-deployment --context='${K8_CONTEXT_DEVELOPMENT}';";
                        sh "kubectl get deployment/blackbox-deployment -o wide --context='${K8_CONTEXT_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"
                    }
                }


                stage ("Prometheus Alert Manager") {
                    steps {

                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/alertmanager-deployment prometheus-alertmanager=${REGISTRY}/prom-altermanager:${env.GIT_COMMIT} --context='${K8_CONTEXT_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/alertmanager-deployment --context='${K8_CONTEXT_DEVELOPMENT}';";
                        sh "kubectl get deployment/alertmanager-deployment -o wide --context='${K8_CONTEXT_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"

                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/alertmanager-deployment prometheus-alertmanager=${REGISTRY}/prom-altermanager:${env.GIT_COMMIT} --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/alertmanager-deployment --context='${K8_CONTEXT_BLUE_DEVELOPMENT}';";
                        sh "kubectl get deployment/alertmanager-deployment -o wide --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"
                    }
                }

                stage ("Prometheus To Teams") {
                    steps {

                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/prom2teams-deployment prometheus-prom2teams=${REGISTRY}/prom-toteams:${env.GIT_COMMIT} --context='${K8_CONTEXT_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/prom2teams-deployment --context='${K8_CONTEXT_DEVELOPMENT}';";
                        sh "kubectl get deployment/prom2teams-deployment -o wide --context='${K8_CONTEXT_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"


                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/prom2teams-deployment prometheus-prom2teams=${REGISTRY}/prom-toteams:${env.GIT_COMMIT} --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/prom2teams-deployment --context='${K8_CONTEXT_BLUE_DEVELOPMENT}';";
                        sh "kubectl get deployment/prom2teams-deployment -o wide --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"
                    }
                }

                stage ("Grafana") {
                    steps {

                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/grafana-deployment grafana=${REGISTRY}/grafana:${env.GIT_COMMIT} --context='${K8_CONTEXT_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/grafana-deployment --context='${K8_CONTEXT_DEVELOPMENT}';";
                        sh "kubectl get deployment/grafana-deployment -o wide --context='${K8_CONTEXT_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"

                        // SECURE PRIVATE DOCKER REGISTRY User Overide to pass in secret
                        sh "kubectl set image deployment/grafana-deployment grafana=${REGISTRY}/grafana:${env.GIT_COMMIT} --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' ;";
                        sh "sleep 6;";
                        sh "timeout 180 kubectl rollout status deployment/grafana-deployment --context='${K8_CONTEXT_BLUE_DEVELOPMENT}';";
                        sh "kubectl get deployment/grafana-deployment -o wide --context='${K8_CONTEXT_BLUE_DEVELOPMENT}' | grep ${env.GIT_COMMIT};"
                    }
                }
            }   
        }
    }
    post {

        success {
            dir("${env.WORKSPACE}") {

            }
            echo 'Successfully Pushed. Updating Grafana User Preferences'
            
        }
        failure {
             dir("${env.WORKSPACE}") {
            }
            echo 'Something Failed'

        }

        cleanup {
            echo 'Cleaning Up Workspace'
            sh "docker images -q --filter=reference=*/prom-monitor | xargs --no-run-if-empty docker rmi -f;"
            sh "docker images -q --filter=reference=*/prom-blackbox | xargs --no-run-if-empty docker rmi -f;"
            sh "docker images -q --filter=reference=*/prom-altermanager | xargs --no-run-if-empty docker rmi -f;"
            sh "docker images -q --filter=reference=*/prom-toteams | xargs --no-run-if-empty docker rmi -f;"
			sh "docker image prune -f;"
            deleteDir()
        }
    }
}


