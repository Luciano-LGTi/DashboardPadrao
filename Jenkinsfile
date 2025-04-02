pipeline {
    agent any

    environment {
        GRAFANA_URL = 'http://grafana:3000'
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
                GRAFANA_URL = 'http://grafana:3000'
            }
            steps {
                withCredentials([string(credentialsId: 'grafana-api-key', variable: 'API_KEY')]) {
                    script {
                        echo '🚀 Iniciando publicação dos dashboards...'

                        def arquivos = findFiles(glob: '**/*.json')
                        echo "📊 Dashboards encontrados: ${arquivos.size()}"

                        arquivos.each { arquivo ->
                            echo "📤 Enviando: ${arquivo.path}"

                            def conteudo = readFile(file: arquivo.path)

                            def payload = [
                                dashboard: new groovy.json.JsonSlurperClassic().parseText(conteudo),
                                overwrite: true,
                                folderUid: null,
                                message: "Importado via Jenkins pipeline"
                            ]

                            def response = httpRequest(
                                httpMode: 'POST',
                                url: "${GRAFANA_URL}/api/dashboards/import",
                                customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                                contentType: 'APPLICATION_JSON',
                                requestBody: groovy.json.JsonOutput.toJson(payload),
                                validResponseCodes: '100:399'
                            )

                            echo "✅ Dashboard '${arquivo.name}' publicado com status: ${response.status}"
                        }
                    }
                }
            }
        }
    }
}
