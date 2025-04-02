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
