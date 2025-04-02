pipeline {
    agent any

    environment {
        GRAFANA_URL = 'http://grafana:3000'
    }

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Clonar repositório') {
            steps {
                echo "🌀 Clonando o repositório com dashboards..."
                git url: 'https://github.com/Luciano-LGTi/DashboardPadrao.git'
            }
        }

        stage('Publicar dashboards no Grafana') {
            environment {
                DASHBOARD_PATH = './'
            }
            steps {
                withCredentials([string(credentialsId: 'grafana-api-key', variable: 'API_KEY')]) {
                    script {
                        echo "🚀 Iniciando publicação dos dashboards..."

                        def files = findFiles(glob: '**/*.json')

                        files.each { file ->
                            def path = file.path.replace('\\', '/')
                            def content = readFile(path)
                            def json = new groovy.json.JsonSlurperClassic().parseText(content)

                            def folderParts = path.tokenize('/')
                            folderParts.pop() // Remove o nome do arquivo
                            def folderName = folderParts.join(' / ').trim()
                            def dashboardTitle = json.dashboard.title ?: file.name

                            echo "📤 Dashboard: ${dashboardTitle}"
                            echo "📁 Pasta: ${folderName}"

                            // Cria pasta se necessário
                            def folderUid = folderName.toLowerCase().replaceAll('[^a-z0-9]', '-')
                            def createFolderResponse = httpRequest(
                                httpMode: 'POST',
                                url: "${GRAFANA_URL}/api/folders",
                                customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                                contentType: 'APPLICATION_JSON',
                                requestBody: """{
                                    "uid": "${folderUid}",
                                    "title": "${folderName}"
                                }""",
                                validResponseCodes: '200:409' // 409 = já existe
                            )
                            echo "📂 Pasta '${folderName}' verificada/criada"

                            // Remove ID para evitar conflitos
                            if (json.dashboard.containsKey('id')) {
                                json.dashboard.remove('id')
                            }

                            def requestBody = [
                                dashboard : json.dashboard,
                                folderUid : folderUid,
                                overwrite : true
                            ]

                            def dashboardJson = groovy.json.JsonOutput.toJson(requestBody)

                            def response = httpRequest(
                                httpMode: 'POST',
                                url: "${GRAFANA_URL}/api/dashboards/db",
                                contentType: 'APPLICATION_JSON',
                                customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                                requestBody: dashboardJson,
                                validResponseCodes: '200:399'
                            )

                            echo "✅ Dashboard '${file.name}' publicado com status: ${response.status}"
                        }
                    }
                }
            }
        }
    }
}
