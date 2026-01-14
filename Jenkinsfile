pipeline {
agent any
stages {
stage ('test') {
bat './gradlew test'
}
stage ('build') { // la phase build
steps { // les étapes de la phase build
bat './gradlew build'

}

}
}

}