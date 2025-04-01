pipeline {
    agent any

    environment {
        GRAFANA_URL = 'http://185.250.36.83:3000'
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
                DASHBOARD_FOLDER = '.' // ou ajuste se estiver em subpasta
            }
            steps {
                withCredentials([string(credentialsId: 'GRAFANA_API_KEY', variable: 'API_KEY')]) {
                    script {
                        echo "🚀 Iniciando publicação dos dashboards..."
                        def files = findFiles(glob: "${env.DASHBOARD_FOLDER}/**/*.json")
                        echo "🔎 Dashboards encontrados: ${files.size()}"

                        for (file in files) {
                            echo "📤 Enviando: ${file.path}"

                            def json = readFile(file.path)
                            def authHeader = "Bearer ${API_KEY}".toString()

                            def response = httpRequest(
                                httpMode: 'POST',
                                url: "${GRAFANA_URL}/api/dashboards/db",
                                customHeaders: [[name: 'Authorization', value: authHeader]],
                                contentType: 'APPLICATION_JSON',
                                requestBody: json,
                                validResponseCodes: '200:299'
                            )

                            echo "✅ Dashboard enviado com sucesso: ${file.name} (Status: ${response.status})"
                        }
                    }
                }
            }
        }
    }
}
