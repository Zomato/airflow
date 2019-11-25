pipeline {
    agent { label "production" }

    stages {
        stage("Bundle") {
            steps {
                withAWS(profile:"JUMBO-ACCOUNT") {
                    script {
                        def branch = env.GIT_BRANCH.replaceAll("(.*)/", "")
                        def webserver_config
                        def airflow_cfg
                        if (branch == "zmaster") {
                            webserver_config = 'webserver_config-prod.py'
                            airflow_cfg = 'airflow-prod.cfg'
                        } else if (branch == "zstaging") {
                            webserver_config = 'webserver_config-staging.py'
                            airflow_cfg = 'airflow-staging.cfg'
                        } else {
                            webserver_config = 'webserver_config-dev.py'
                            airflow_cfg = 'airflow-dev.cfg'
                        }
                        sh "docker build -t zdp-airflow:${branch} --build-arg WEBSERVER_CONFIG=${webserver_config} \
                        --build-arg AIRFLOW_CFG=${airflow_cfg} ."
                    }
                }
            }
        }

        stage("Version") {
            steps {
                withAWS(profile:"JUMBO-ACCOUNT") {
                    script {
                        def branch = env.GIT_BRANCH.replaceAll("(.*)/", "")
                        def tag = "${branch}_${GIT_COMMIT}"
                        sh "\$(aws ecr get-login --no-include-email --region ap-southeast-1)"
                        sh "docker tag zdp-airflow:${branch} 125719378300.dkr.ecr.ap-southeast-1.amazonaws.com/zdp-airflow:${tag}"
                        sh "docker push 125719378300.dkr.ecr.ap-southeast-1.amazonaws.com/zdp-airflow:${tag}"
                    }
                }
            }
        }

        stage("Deploy") {
            steps {
                withAWS(profile:"JUMBO-ACCOUNT") {
                    script {
                        def branch = env.GIT_BRANCH.replaceAll("(.*)/", "")
                        sh "\$(aws ecr get-login --no-include-email --region ap-southeast-1)"
                        sh "docker tag zdp-airflow:${branch} 125719378300.dkr.ecr.ap-southeast-1.amazonaws.com/zdp-airflow:${branch}"
                        sh "docker push 125719378300.dkr.ecr.ap-southeast-1.amazonaws.com/zdp-airflow:${branch}"
                    }
                }
            }
        }

        stage("Clean") {
            steps {
                withAWS(profile:"JUMBO-ACCOUNT") {
                    script {
                        sh "docker system prune -f"
                    }
                }
            }
        }
    }
}
