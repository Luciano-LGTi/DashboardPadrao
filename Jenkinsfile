pipeline {
    agent any

    environment {
        DASHBOARDS_DIR = '.'
    }

    parameters {
        password(name: 'GRAFANA_API_KEY', defaultValue: '', description: 'Chave da API do Grafana')
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
                GRAFANA_URL = 'http://grafana:3000/api/dashboards/import'
            }
            steps {
                withCredentials([string(credentialsId: 'grafana-api-key', variable: 'API_KEY')]) {
                    script {
                        echo '🚀 Iniciando publicação dos dashboards...'
                        def files = findFiles(glob: '**/*.json')

                        echo "📊 Dashboards encontrados: ${files.length}"

                        files.each { file ->
                            echo "📤 Enviando: ${file.path}"
                            def jsonContent = readFile(file.path).trim()

                            def payload = """
                            {
                                "dashboard": ${jsonContent},
                                "overwrite": true,
                                "message": "Importado via Jenkins pipeline"
                            }
                            """

                            def response = httpRequest(
                                httpMode: 'POST',
                                url: GRAFANA_URL,
                                contentType: 'APPLICATION_JSON',
                                customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                                requestBody: payload,
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
