pipeline {
    agent any

    parameters {
        string(name: 'GRAFANA_URL', defaultValue: 'http://185.250.36.83:3000', description: 'URL da instância Grafana')
        credentials(name: 'grafana-api-key', description: 'Token do Grafana')
    }

    environment {
        API_KEY = "${params.GRAFANA_API_KEY}"
    }

    stages {
        stage('Clonar repositório') {
            steps {
                git branch: 'main', url: 'https://github.com/Luciano-LGTi/DashboardPadrao.git'
            }
        }

        stage('Publicar dashboards') {
            steps {
                script {
                    def dashboards = findFiles(glob: '**/*.json')
                    for (file in dashboards) {
                        echo "Publicando: ${file.path}"
                        def json = readFile(file.path)

                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "${params.GRAFANA_URL}/api/dashboards/db",
                            contentType: 'APPLICATION_JSON',
                            customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                            requestBody: """
                            {
                                "dashboard": ${json},
                                "overwrite": true,
                                "folderId": 0
                            }
                            """
                        )

                        echo "Dashboard '${file.name}' publicado com status: ${response.status}"
                    }
                }
            }
        }
    }
}
