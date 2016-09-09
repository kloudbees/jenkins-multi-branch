def jettyUrl = 'http://localhost:8081/'

stage 'Dev'
node ('node-cd'){
    checkout scm
    // mvn '-o clean package'
    // archive 'target/x.war'
}

stage 'QA'
parallel(longerTests: {
    runTests(30)
}, quickerTests: {
    runTests(20)
})

stage name: 'Staging', concurrency: 1
node ('node-cd'){
    deploy 'staging'
}

input message: "Does staging look good?"
try {
    checkpoint('Before production')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}

stage name: 'Production', concurrency: 1
node ('node-production') {
    //sh "wget -O - -S ${jettyUrl}staging/"
    echo 'Production server looks to be alive'
    deploy 'production'
    echo "Deployed to ${jettyUrl}production/"
}

def mvn(args) {
    sh "${tool 'Maven 3.x'}/bin/mvn ${args}"
}

def runTests(duration) {
    node ('node-cd'){
        checkout scm
        sh "sleep ${duration}"
        /*
        runWithServer {url ->
            mvn "-o -f sometests test -Durl=${url} -Dduration=${duration}"
        } */
    }
}

def deploy(id) {
    // unarchive mapping: ['target/x.war' : 'x.war']
    // sh "cp x.war /tmp/webapps/${id}.war"
    sh "sleep 10"
}

def undeploy(id) {
   // sh "rm /tmp/webapps/${id}.war"
}

def runWithServer(body) {
    def jettyUrl = 'http://localhost:8081/' // TODO why is this not inherited from the top-level scope?
    def id = UUID.randomUUID().toString()
    deploy id
    
    try {
        body.call "${jettyUrl}${id}/"
    } finally {
        undeploy id
    }
}
