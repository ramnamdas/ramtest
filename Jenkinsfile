Helmdeploy.groovy


def call(body) {
    // evaluate the body block, and collect configuration into the object
    def config = [:]
    body.resolveStrategy = Closure.DELEGATE_FIRST
    body.delegate = config
    body()
    repoHelmCharts = config.get("repoHelmCharts", "git")
    setValues = config.get("setValues", "")
    helmChartName = commonLib.getHelmChartName()
    helmRepoName = commonLib.getHelmRepoName()
    def hasIngress = config.hasIngress
    node("tools03") {
        if (helmChartName == "") {
            chartName = "${env.JOB_NAME}".split("/").last()
        }
        printf("Chart name is %s", helmChartName)
        if (repoHelmCharts == "git") {
            stage('Pull Helm Charts') {
                scmVars = checkout scm
            }
        } else if (repoHelmCharts == "artifactory") {
            stage("Update Helm from Artifactory") {
                sh "helm repo add ${helmRepoName} https://arti.lululemon.app:443/artifactory/${helmRepoName}/"
                sh "helm repo update"
                sh "helm fetch ${helmRepoName}/${helmChartName} --untar"
            }
        }
        withFolderProperties {
            def namespace = commonLib.getTargetNamespace()
            def is_external_docker = env.isexternaldocker
            println("is_external_docker is ${is_external_docker}")
            def envr = env.environment
            def chartName = env.JOB_NAME.split("/").last()
            def dockerRepo = null
            if (is_external_docker != null && is_external_docker != "true") {
                dockerRepo = eksUtilities.getDockerRegistry(chartName)
            }
            println("dockerRepo is ${dockerRepo}")
            def releaseName = chartName + "-${envr}"
            def extraOpts = "--values values/${envr}.yaml"
            if (env.setValues != null) {
                for (setValue in env.setValues.split()) {
                    setValues = setValues + " --set " + setValue
                }
            }
            if ((envr == "prod") || (envr == "prd") || (envr == "stg")) {
                if ((envr == "prod") || (envr == "prd")) {
                    namespace = env.projectname
                }
            }
            println("SetValues variable = ${setValues}")
            if (is_external_docker != null && is_external_docker == "true") {
                eksUtilities.runExternalHelmDeployWithValues(envr, namespace, releaseName, chartName, extraOpts)
            } else {
                eksUtilities.runHelmDeploy(envr, namespace, releaseName, chartName, setValues, extraOpts, hasIngress)
            }
        }
    }
}



