pipeline {
    agent any

    parameters {
        string(name: 'GRAFANA_URL', defaultValue: 'http://grafana:3000', description: 'URL da instância Grafana')
        string(name: 'API_KEY', description: 'API Token Grafana com permissão de administrador')
        string(name: 'ORG_ID', defaultValue: '1', description: 'ID da organização no Grafana')
    }

    stages {
        stage('Clonar repositório') {
            steps {
                echo '🌠 Clonando o repositório com dashboards...'
                git branch: 'main', url: 'https://github.com/Luciano-LGTi/DashboardPadrao.git'
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

                        json?.templating?.list?.each { template ->
                            if (template.datasource) {
                                def ds = template.datasource instanceof Map ? template.datasource.uid : template.datasource
                                def dstype = template.type ?: 'mysql'
                                datasources[ds] = dstype
                            }
                        }

                        json?.panels?.each { panel ->
                            panel?.targets?.each { target ->
                                if (target.datasource) {
                                    def ds = target.datasource instanceof Map ? target.datasource.uid : target.datasource
                                    def dstype = target.type ?: 'mysql'
                                    datasources[ds] = dstype
                                }
                            }
                        }
                    }

                    echo "🔧 Datasources identificados: ${datasources}"

                    datasources.each { ds, dstype ->
                        def responseGet = httpRequest(
                            httpMode: 'GET',
                            url: "${params.GRAFANA_URL}/api/datasources/name/${ds}",
                            customHeaders: [
                                [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                            ],
                            validResponseCodes: '100:404'
                        )

                        if (responseGet.status == 404) {
                            def requestBody = """{
                                \"name\": \"${ds}\",
                                \"type\": \"${dstype}\",
                                \"access\": \"proxy\",
                                \"url\": \"http://${ds}.local\",
                                \"database\": \"${ds}_db\",
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

                            echo "✅ Datasource '${ds}' criado com status: ${responseCreate.status}"
                        } else {
                            echo "ℹ️ Datasource '${ds}' já existe."
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
                    echo "📊 Dashboards encontrados: ${dashboards.size()}"
                }
            }
        }
    }
}
