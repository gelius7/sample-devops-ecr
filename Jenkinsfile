def SERVICE_GROUP = "sample"
def SERVICE_NAME = "devops"
def IMAGE_NAME = "${SERVICE_GROUP}-${SERVICE_NAME}"
def REPOSITORY_URL = "https://github.com/gelius7/sample-devops-ecr.git"
def REPOSITORY_SECRET = ""
def SLACK_TOKEN_DEV = ""
def SLACK_TOKEN_DQA = ""

//@Library("github.com/opsnow-tools/valve-butler")
@Library("github.com/gelius7/valve-butler")
def butler = new com.opsnow.valve.v7.Butler()
def label = "worker-${UUID.randomUUID().toString()}"

properties([
  buildDiscarder(logRotator(daysToKeepStr: "60", numToKeepStr: "30"))
])
podTemplate(label: label, containers: [
  containerTemplate(name: "builder", image: "opsnowtools/valve-builder:v0.2.2", command: "cat", ttyEnabled: true, alwaysPullImage: true),
  containerTemplate(name: "node", image: "node:10", command: "cat", ttyEnabled: true)
], volumes: [
  hostPathVolume(mountPath: "/var/run/docker.sock", hostPath: "/var/run/docker.sock"),
  hostPathVolume(mountPath: "/home/jenkins/.draft", hostPath: "/home/jenkins/.draft"),
  hostPathVolume(mountPath: "/home/jenkins/.helm", hostPath: "/home/jenkins/.helm")
]) {
  node(label) {
    stage("Prepare") {
      container("builder") {
        butler.prepare(IMAGE_NAME)
      }
    }
    stage("Checkout") {
      container("builder") {
        try {
          git(url: REPOSITORY_URL, branch: BRANCH_NAME)
        } catch (e) {
          butler.failure(SLACK_TOKEN_DEV, "checkout")
            throw e
        }

        butler.scan("nodejs")
      }
    }
    if (BRANCH_NAME == "master") {
      stage("Build Charts") {
        container("builder") {
          try {
            butler.build_chart()
          } catch (e) {
            butler.failure(SLACK_TOKEN_DEV, "Build Charts")
            throw e
          }
        }
      }
      stage("Deploy DEV") {
        container("builder") {
          try {
            // deploy(cluster, namespace, sub_domain, profile)
            butler.deploy("here", "${SERVICE_GROUP}-dev", "${IMAGE_NAME}-dev", "dev")
            butler.success(SLACK_TOKEN_DEV, "Deploy DEV")
          } catch (e) {
            butler.failure(SLACK_TOKEN_DEV, "Deploy DEV")
            throw e
          }
        }
      }
    }
  }
}
