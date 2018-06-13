#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def SCRATCH_ALIAS = 'TDX18'

    def toolbelt = tool 'toolbelt'

    stage('Checkout Source from VCS') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
    	stage('Authenticate Devhub') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\"sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            if (rc != 0) { error 'hub org authorization failed' }
        }
        
        stage('Create Scratch Org') {
            // need to pull out assigned username
            //rmsg = sh returnStdout: true, script: "\"${toolbelt}\"sfdx force:org:create -f config/developerOrg-scratch-def.json --json -s -a df13@makepositive.com"
            rmsg = bat returnStdout: true, script: "\"${toolbelt}\"sfdx force:org:create -f config/developerOrg-scratch-def.json --json -s -a ${SCRATCH_ALIAS} --durationdays 1"

	    println(rmsg)
		
            //def jsonSlurper = new JsonSlurperClassic()
            //def robj = jsonSlurper.parseText(rmsg)
            //if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            //SFDC_USERNAME=robj.result.username
	    SFDC_USERNAME='test-ztb7oxipfmri@example.com'
            println(SFDC_USERNAME)
	    SFDC_USERNAME="${SCRATCH_ALIAS}"	
            //robj = null

	    rc = bat returnStatus: true, script: "\"${toolbelt}\"sfdx force:config:set --global defaultusername=${SFDC_USERNAME} --json"
            if (rc != 0) { error 'Default scratch org failed' }
		
        }

	// Combined into create scratch org 
    	//stage('Set Default scratch org') {
        //    rc = bat returnStatus: true, script: "\"${toolbelt}\"sfdx force:config:set --global defaultusername=${SFDC_USERNAME} --json"
        //    if (rc != 0) { error 'Default scratch org failed' }
        //}

        stage('Create Scratch Org Password') {
 			rmsg = bat returnStdout: true, script: "\"${toolbelt}\"sfdx force:user:password:generate --json"
			println(rmsg)
			//def jsonSlurper = new JsonSlurperClassic()
			//def robj = jsonSlurper.parseText(rmsg)
            //if (robj.status != 0) { error 'password generation failed: ' + robj.message }
            //robj = null
	    SFDC_PASSWORD='[%MuIS43_t'	
        }
		
        stage('Push to Test Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\"sfdx force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) { error 'Push failed'}
		
	    // Grant DH Permission set
	    rc = bat returnStatus: true, script: "\"${toolbelt}\"sfdx  force:user:permset:assign --permsetname DreamHouse --targetusername ${SFDC_USERNAME}"
            if (rc != 0) { error 'Permset Assignment failed'}
		
        }

        stage('Run Apex Unit Tests') {
            powershell "mkdir -p ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\"sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                   // error 'Apex test run failed'
                }
            }
        }	    
	    
        //stage('Create Users in Scratch Org') {
		//	rc = sh returnStatus: true, script: "\"${toolbelt}\"sfdx force:user:create -a scratchOrg2@user2.com -f config/user-scratch-def.json --json --targetusername ${SFDC_USERNAME}"
        //    if (rc != 0) {
        //        error 'User creation failed'
        //    }
        //    
        //    rc = sh returnStatus: true, script: "\"${toolbelt}\"sfdx force:user:create -a scratchOrg2@user2.com -f config/user-scratch-def.json --json profileName="Chatter Free User" --targetusername ${SFDC_USERNAME}"
        //    if (rc != 0) {
        //        error 'User creation failed'
        //    }
        //}

	stage('Load Test Data') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\"sfdx force:data:tree:import --plan data/sample-data-plan.json"
            if (rc != 0) { error 'Push failed'}

	    println('Dreamhouse test data imported')
	}
        stage('Run Provar Test Cases') {
		SFDC_USERNAME = 'test-ztb7oxipfmri@example.com'
	    	println(SFDC_USERNAME)
	    	//rmsg = bat returnStdout: true, script: "ant -f webinar/ANT/build.xml -DSFDC_USERNAME_SO=${SFDC_USERNAME}"
	    	rmsg = bat returnStdout: true, script: "ant -f webinar/ANT/build.xml -DSFDC_USERNAME_SO=${SFDC_USERNAME}"
	    	//rmsg = bat returnStdout: true, script: "ant -f c:/Users/ProvarTrial4/Provar/StandardDemo/WebinarDemo/ANT/build.xml"
		
	        println(rmsg)
	    }

        //stage('Run Apex Test') {
        //    sh "mkdir -p ${RUN_ARTIFACT_DIR}"
        //    timeout(time: 120, unit: 'SECONDS') {
        //        rc = sh returnStatus: true, script: "\"${toolbelt}\"sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
        //        if (rc != 0) {
        //            error 'apex test run failed'
        //        }
        //    }
        //}

        stage('Publish Junit Test Results') {
            junit keepLongStdio: true, testResults: 'webinar/ANT/**/*.xml'
        }
    }
}
