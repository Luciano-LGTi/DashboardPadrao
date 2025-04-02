pipeline {
    agent any

    parameters {
        string(name: 'GRAFANA_URL', defaultValue: 'http://grafana:3000', description: 'URL da instância Grafana')
        credentials(name: 'grafana-api-key', description: 'Token do Grafana (ID da credencial deve ser grafana-api-key)')
        string(name: 'ORG_ID', defaultValue: '1', description: 'ID da organização no Grafana')
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

                        def rawJson = readFile(file.path)
                        def jsonStr = removeIdField(rawJson)

                        def requestBody = """{
                            "dashboard": ${jsonStr},
                            "overwrite": true,
                            "folderId": 0
                        }"""

                        echo "📄 JSON parcial:"
                        echo requestBody.take(1000)

                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "${params.GRAFANA_URL}/api/dashboards/import",
                            contentType: 'APPLICATION_JSON',
                            customHeaders: [
                                [name: 'Authorization', value: "Bearer ${API_KEY}"],
                                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                            ],
                            requestBody: requestBody
                        )

                        echo "✅ Dashboard '${file.name}' publicado com status: ${response.status}"
                    }
                }
            }
        }
    }
}

// Função fora do pipeline para evitar LazyMap no histórico
@NonCPS
def removeIdField(rawJson) {
    def parser = new groovy.json.JsonSlurperClassic()
    def obj = parser.parseText(rawJson)
    obj.remove('id')
    return groovy.json.JsonOutput.toJson(obj)
}
