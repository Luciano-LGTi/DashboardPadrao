pipeline {
    agent any

    environment {
        GRAFANA_URL = 'http://grafana:3000'
        FOLDER_PATH = '.' // Raiz do repositório
    }

    stages {
        stage('Clonar repositório') {
            steps {
                echo '🌀 Clonando o repositório com dashboards...'
                git url: 'https://github.com/Luciano-LGTi/DashboardPadrao.git', branch: 'main'
            }
        }

        stage('Publicar dashboards no Grafana') {
            environment {
                CONTENT_TYPE = 'application/json'
            }
            steps {
                withCredentials([string(credentialsId: 'grafana-api-key', variable: 'API_KEY')]) {
                    script {
                        echo '🚀 Iniciando publicação dos dashboards...'

                        def arquivos = findFiles(glob: '**/*.json')
                        echo "📊 Dashboards encontrados: ${arquivos.size()}"

                        arquivos.each { file ->
                            echo "📤 Enviando: ${file.path}"

                            def rawJson = readFile(file.path)
                            def json = new groovy.json.JsonSlurperClassic().parseText(rawJson)

                            // Remove o campo "id", se existir
                            if (json.dashboard?.id != null) {
                                json.dashboard.remove('id')
                            }

                            def payload = [
                                dashboard: json.dashboard,
                                overwrite: true,
                                folderUid: null, // Pode personalizar se quiser enviar pra pastas específicas
                                message: "Importado via Jenkins pipeline"
                            ]

                            def response = httpRequest(
                                httpMode: 'POST',
                                url: "${GRAFANA_URL}/api/dashboards/import",
                                customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                                contentType: CONTENT_TYPE,
                                requestBody: groovy.json.JsonOutput.toJson(payload),
                                validResponseCodes: '100:399'
                            )

                            echo "✅ Dashboard '${file.name}' publicado com status: ${response.status}"
                        }
                    }
                }
            }
        }
    }
}
