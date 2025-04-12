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
                    def datasources = []

                    dashboards.each { file ->
                        def rawJson = readFile(file.path)
                        def json = new groovy.json.JsonSlurperClassic().parseText(rawJson)

                        json.panels?.each { panel ->
                            if (panel.datasource) {
                                def datasourceName = panel.datasource instanceof Map ? panel.datasource.uid : panel.datasource
                                def datasourceType = panel.type ?: 'prometheus'

                                if (!datasources.find { it.name == datasourceName && it.type == datasourceType }) {
                                    datasources << [name: datasourceName, type: datasourceType]
                                }
                            }
                        }
                    }

                    echo "🔧 Datasources identificados: ${datasources}"

                    datasources.each { ds ->
                        def responseGet = httpRequest(
                            httpMode: 'GET',
                            url: "${params.GRAFANA_URL}/api/datasources/uid/${ds.name}",
                            customHeaders: [
                                [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                            ],
                            validResponseCodes: '100:404'
                        )

                        if (responseGet.status == 404) {
                            def requestBody = """{
                                \"name\": \"${ds.name} - ${ds.type}\",
                                \"type\": \"${ds.type}\",
                                \"access\": \"proxy\",
                                \"url\": \"http://localhost\"
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

                            echo "✅ Datasource '${ds.name}' criado com status: ${responseCreate.status}"
                        } else {
                            echo "ℹ️ Datasource '${ds.name}' já existe."
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

                    dashboards.each { file ->
                        def folderPath = file.path.tokenize('/')[0..-2]
                        def rawJson = readFile(file.path)
                        def jsonStr = removeIdField(rawJson)

                        def folderId = getOrCreateFolder(folderPath)

                        def requestBody = """{
                            \"dashboard\": ${jsonStr},
                            \"overwrite\": true,
                            \"folderId\": ${folderId}
                        }"""

                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "${params.GRAFANA_URL}/api/dashboards/import",
                            contentType: 'APPLICATION_JSON',
                            customHeaders: [
                                [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                            ],
                            requestBody: requestBody
                        )

                        echo "✅ Dashboard '${file.name}' publicado com status: ${response.status} na pasta '${folderPath.join('/')}'"
                    }
                }
            }
        }
    }
}

@NonCPS
def removeIdField(rawJson) {
    def parser = new groovy.json.JsonSlurperClassic()
    def obj = parser.parseText(rawJson)
    obj.remove('id')
    return groovy.json.JsonOutput.toJson(obj)
}


def getOrCreateFolder(folderPath) {
    def folderFullPath = folderPath.join(' - ')
    def response = httpRequest(
        httpMode: 'GET',
        url: "${params.GRAFANA_URL}/api/folders",
        customHeaders: [
            [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
            [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
        ]
    )

    def folders = new groovy.json.JsonSlurperClassic().parseText(response.content)
    def folder = folders.find { it.title == folderFullPath }

    if (folder) {
        return folder.id
    } else {
        def createResponse = httpRequest(
            httpMode: 'POST',
            url: "${params.GRAFANA_URL}/api/folders",
            contentType: 'APPLICATION_JSON',
            customHeaders: [
                [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
            ],
            requestBody: """{
                \"title\": \"${folderFullPath}\"
            }"""
        )

        def createdFolder = new groovy.json.JsonSlurperClassic().parseText(createResponse.content)
        return createdFolder.id
    }
}
