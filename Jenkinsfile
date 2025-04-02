""pipeline {
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
                API_KEY = credentials('grafana-api-key')
            }
            steps {
                script {
                    echo "🚀 Iniciando publicação dos dashboards..."

                    def files = findFiles(glob: '**/*.json')
                    echo "📊 Dashboards encontrados: ${files.size()}"

                    files.each { file ->
                        echo "📤 Enviando: ${file.path}"

                        def content = readFile(file.path)

                        // Detectar pasta
                        def pathParts = file.path.tokenize('/')
                        def folderName = pathParts.size() > 1 ? pathParts[0..-2].join(' / ') : 'General'

                        echo "📁 Pasta detectada: ${folderName}"

                        def dashboardJson = readJSON text: content
                        def payload = groovy.json.JsonOutput.toJson([
                            dashboard: dashboardJson,
                            overwrite: true,
                            folderUid: null, // ou defina um UID se quiser organizar em pastas fixas
                            message: "Importado via Jenkins pipeline"
                        ])

                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "${GRAFANA_URL}/api/dashboards/import",
                            customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                            contentType: 'APPLICATION_JSON',
                            requestBody: payload,
                            validResponseCodes: '100:399'
                        )

                        echo "✅ Dashboard '${file.name}' publicado com status: ${response.statusCode}"
                    }
                }
            }
        }
    }
}""