#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

	//stage('Authenticate dehub user') {
	//    rc = sh returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
	//    if (rc != 0) { error 'hub org authorization failed' }
	//}

    stage('Create Scratch Org') {
        rmsg = sh returnStdout: true, script: "\"${toolbelt}\" force:org:create -f config/developerOrg-scratch-def.json --json -s -a ScratchOrg1"
        println(rmsg)
        def jsonSlurper = new JsonSlurperClassic()
        def robj = jsonSlurper.parseText(rmsg)
        if (robj.status != 0) { error 'org creation failed: ' + robj.message }
        SFDC_USERNAME=robj.result.username
        env.SFDC_USERNAME_SO = SFDC_USERNAME
        println(SFDC_USERNAME)
        println(env.SFDC_USERNAME_SO)
        robj = null
    }

    stage('Create password for scratch org') {
		rmsg = sh returnStdout: true, script: "\"${toolbelt}\" force:user:password:generate --json"
		println(rmsg)
		def jsonSlurper = new JsonSlurperClassic()
		def robj = jsonSlurper.parseText(rmsg)
        if (robj.status != 0) { error 'password generation failed: ' + robj.message }
        robj = null
    }
	
    stage('Push To Test Org') {
        rc = sh returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
        if (rc != 0) { error 'Push failed'}
    }
    
    stage('Create Users in scratch org') {
		rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:create -a scratchOrg2@user2.com -f config/user-scratch-def.json --json profileName="Standard User" --targetusername ${SFDC_USERNAME}"
        if (rc != 0) {
            error 'User creation failed'
        }
        rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:create -a scratchOrg2@user2.com -f config/user-scratch-def.json --json profileName="Chatter Free User" --targetusername ${SFDC_USERNAME}"
        if (rc != 0) {
            error 'User creation failed'
        }
    }

    //stage('Run Provar test cases') {
	//	rc = sh returnStatus: true, script: "ant -f build.xml -DadminUser=${SFDC_USERNAME}"
    //    if (rc != 0) {
    //        error 'User creation failed'
    //    }
    //}

    //stage('collect results') {
    //    junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
    //}
}
