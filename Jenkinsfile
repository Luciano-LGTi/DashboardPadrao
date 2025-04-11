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
                echo '🌀 Clonando o repositório com dashboards...'
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

                        json?.templating?.list?.each { item ->
                            if (item.type == 'datasource' && item.query && !datasources.contains(item.query)) {
                                datasources << item.query
                            }
                        }
                    }

                    echo "🔧 Datasources identificados: ${datasources}"

                    datasources.each { datasource ->
                        def requestBody = """{
                            \"name\": \"${datasource}\",
                            \"type\": \"prometheus\",
                            \"access\": \"proxy\",
                            \"url\": \"http://localhost:9090\"
                        }"""

                        def response = httpRequest(
                            httpMode: 'POST',
                            url: "${params.GRAFANA_URL}/api/datasources",
                            contentType: 'APPLICATION_JSON',
                            customHeaders: [
                                [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
                                [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
                            ],
                            requestBody: requestBody
                        )

                        echo "✅ Datasource '${datasource}' criado com status: ${response.status}"
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

                        echo "✅ Dashboard '${file.name}' publicado com status: ${response.status} na pasta '${folderPath.join('/')}""
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
