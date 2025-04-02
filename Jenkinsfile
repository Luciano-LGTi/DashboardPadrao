pipeline {
    agent any

    parameters {
        string(name: 'GRAFANA_URL', defaultValue: 'http://grafana:3000', description: 'URL da instância Grafana')
        credentials(name: 'grafana-api-key', description: 'Token do Grafana')
    }

    environment {
        API_KEY = credentials('grafana-api-key')
    }

    stages {
        stage('Clonar repositório') {
            steps {
                echo '🌀 Clonando o repositório com dashboards...'
                git branch: 'main', url: 'https://github.com/Luciano-LGTi/DashboardPadrao.git'
            }
        }

        stage('Publicar dashboards no Grafana') {
            steps {
                script {
                    echo '🚀 Iniciando publicação dos dashboards...'
                    def dashboards = findFiles(glob: '**/*.json')
                    echo "📊 Dashboards encontrados: ${dashboards.size()}"

                    for (file in dashboards) {
                        echo "📤 Enviando: ${file.path}"
                        def dashboardJsonRaw = readFile(file.path)
                        def parsedJson = new groovy.json.JsonSlurper().parseText(dashboardJsonRaw)
                        def dashboardEscaped = groovy.json.JsonOutput.toJson(parsedJson)

                        def requestPayload = """{
                          "dashboard": ${dashboardEscaped},
                          "overwrite": true,
                          "folderId": 0
                        }"""

                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "${params.GRAFANA_URL}/api/dashboards/import",
                            contentType: 'APPLICATION_JSON',
                            customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                            requestBody: requestPayload
                        )

                        echo "✅ Dashboard '${file.name}' publicado com status: ${response.status}"
                    }
                }
            }
        }
    }
}
