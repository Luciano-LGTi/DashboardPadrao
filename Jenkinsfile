pipeline {
    agent any

    parameters {
        string(name: 'GRAFANA_URL', defaultValue: 'http://grafana:3000', description: 'URL da instância Grafana')
        string(name: 'API_KEY', description: 'API Token Grafana com permissão de administrador')
        string(name: 'ORG_ID', defaultValue: '1', description: 'ID da organização no Grafana')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch do repositório')
    }

    stages {
        stage('Clonar repositório') {
            steps {
                echo '🌠 Clonando o repositório com dashboards...'
                git branch: "${params.BRANCH}", url: 'https://github.com/Luciano-LGTi/DashboardPadrao.git'
            }
        }

        stage('Identificar e Criar Datasources') {
            steps {
                script {
                    echo '🔍 Identificando datasources nas dashboards...'
                    def dashboards = findFiles(glob: '**/*.json')
                    def datasources = [:]

                    dashboards.each { file ->
                        def rawJson = readFile(file.path)
                        def json = new groovy.json.JsonSlurperClassic().parseText(rawJson)

                        def types = []

                        def extractTypes = { obj ->
                            if (obj instanceof String) {
                                return
                            }
                            if (obj instanceof Map && obj?.type && obj?.type != 'datasource') {
                                types << obj.type
                            }
                        }

                        extractTypes(json?.datasource)

                        json?.templating?.list?.each { template ->
                            extractTypes(template?.datasource)
                        }

                        json?.panels?.each { panel ->
                            extractTypes(panel?.datasource)
                            panel?.targets?.each { target ->
                                extractTypes(target?.datasource)
                            }
                        }

                        types.each { t ->
                            datasources[t] = t
                        }
                    }

                    echo "🔧 Datasources identificados: ${datasources}"

                    datasources.each { ds, dstype ->
                        def responseGet = httpRequest(
                            httpMode: 'GET',
                            url: "${params.GRAFANA_URL}/api/datasources/name/${dstype}",
                            customHeaders: [
                                [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                            ],
                            validResponseCodes: '100:404'
                        )

                        if (responseGet.status == 404) {
                            def requestBody = """{
                                \"name\": \"${dstype}\",
                                \"type\": \"${dstype}\",
                                \"access\": \"proxy\",
                                \"url\": \"http://${dstype}.local\",
                                \"basicAuth\": false,
                                \"isDefault\": false
                            }"""

                            def responseCreate = httpRequest(
                                httpMode: 'POST',
                                url: "${params.GRAFANA_URL}/api/datasources",
                                contentType: 'APPLICATION_JSON',
                                customHeaders: [
                                    [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                                    [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                                ],
                                requestBody: requestBody
                            )

                            echo "✅ Datasource '${dstype}' criado com status: ${responseCreate.status}"
                        } else {
                            echo "ℹ️ Datasource '${dstype}' já existe."
                        }
                    }
                }
            }
        }

        stage('Publicar dashboards no Grafana') {
            steps {
                script {
                    echo '🚀 Iniciando publicação dos dashboards...'
                    def dashboards = findFiles(glob: '**/*.json')

                    dashboards.each { file ->
                        def rawJson = readFile(file.path)

                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "${params.GRAFANA_URL}/api/dashboards/db",
                            contentType: 'APPLICATION_JSON',
                            customHeaders: [
                                [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                            ],
                            requestBody: rawJson
                        )

                        echo "✅ Dashboard '${file.name}' publicado com status: ${response.status}"
                    }
                }
            }
        }
    }
}
