pipeline {
    agent any

    parameters {
        string(name: 'GRAFANA_URL', defaultValue: 'http://grafana:3000', description: 'URL da instância Grafana')
        string(name: 'API_KEY', description: 'API Token Grafana com permissão de administrador')
        string(name: 'ORG_ID', defaultValue: '1', description: 'ID da organização no Grafana')
    }

    stages {
        stage('Preparar Ambiente') {
            steps {
                echo '🏗️ Preparando ambiente...'
                deleteDir()
                checkout scm
            }
        }

        stage('Identificar e Criar Datasources') {
            steps {
                script {
                    echo '🔍 Identificando datasources nas dashboards...'

                    def dashboards = findFiles(glob: '**/*.json')
                    def datasourcesEncontrados = []

                    dashboards.each { file ->
                        def content = readFile(file.path)
                        def json = new groovy.json.JsonSlurperClassic().parseText(content)

                        json.__inputs?.each { input ->
                            if (input.type == 'datasource') {
                                datasourcesEncontrados << [name: input.name, type: input.pluginId ?: 'mysql']
                            }
                        }
                    }

                    datasourcesEncontrados.unique().each { ds ->
                        echo "🔧 Verificando datasource: ${ds.name}"
                        if (!verificarDatasourceExistente(ds.name)) {
                            criarDatasource(ds.name, ds.type)
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

                    def datasources = obterListaDatasources()

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
