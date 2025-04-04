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

        stage('Publicar dashboards no Grafana') {
            steps {
                script {
                    echo '🚀 Iniciando publicação dos dashboards...'
                    def dashboards = findFiles(glob: '**/*.json')
                    echo "📊 Dashboards encontrados: ${dashboards.size()}"

                    def datasources = getDatasources()

                    dashboards.each { file ->
                        def folderPath = file.path.tokenize('/')[0..-2]
                        def rawJson = readFile(file.path)
                        def jsonStr = updateDatasourceIds(removeIdField(rawJson), datasources)

                        def folderId = getOrCreateFolder(folderPath.join(' - '))

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

                        echo "✅ Dashboard '${file.name}' publicado com status: ${response.status} na pasta '${folderPath.join(' - ')}'"
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

def getOrCreateFolder(folderName) {
    def response = httpRequest(
        httpMode: 'GET',
        url: "${params.GRAFANA_URL}/api/folders",
        customHeaders: [
            [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
            [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
        ]
    )

    def folders = new groovy.json.JsonSlurperClassic().parseText(response.content)
    def folder = folders.find { it.title == folderName }

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
                \"title\": \"${folderName}\"
            }"""
        )

        def createdFolder = new groovy.json.JsonSlurperClassic().parseText(createResponse.content)
        return createdFolder.id
    }
}

def getDatasources() {
    def response = httpRequest(
        httpMode: 'GET',
        url: "${params.GRAFANA_URL}/api/datasources",
        customHeaders: [
            [name: 'Authorization', value: "Bearer ${params.API_KEY}"],
            [name: 'X-Grafana-Org-Id', value: "${params.ORG_ID}"]
        ]
    )

    return new groovy.json.JsonSlurperClassic().parseText(response.content)
}

@NonCPS
def updateDatasourceIds(rawJson, datasources) {
    def parser = new groovy.json.JsonSlurperClassic()
    def dashboard = parser.parseText(rawJson)

    dashboard.panels.each { panel ->
        if (panel.datasource && panel.datasource.uid) {
            def matchingDs = datasources.find { it.uid == panel.datasource.uid }
            if (matchingDs) {
                panel.datasource.uid = matchingDs.uid
            }
        }
    }

    return groovy.json.JsonOutput.toJson(dashboard)
}
