#!groovy

properties(
    [
        parameters(
            [
                string(description: 'CI Message', defaultValue: '', name: 'CI_MESSAGE'),
            ]
        ),
	
        pipelineTriggers(
            [
		[
		    $class: 'CIBuildTrigger',
		    noSquash: true,
		    providerData: [
                        $class: 'RabbitMQSubscriberProviderData',
                        name: 'RabbitMQ',
			
                        overrides: [
                            topic: 'org.fedoraproject.prod.buildsys.tag',
                            queue: 'osci-pipelines-queue-12'
                        ],

			checks: [
                            [
                                expectedValue: '^f34$',
                                field: '$.tag'
                            ]
			]
                    ]
                ]
	    ]
        )
    ]
)

def msg

pipeline {

    agent any

    stages {
        stage('Trigger build') {
            steps {
                script {
                    msg = readJSON text: CI_MESSAGE

		    // Example:
		    // {
		    //   "owner":"hobbes1069",
		    //   "instance":"primary",
		    //   "build_id":1530872,
		    //   "release":"1.fc33",
		    //   "name":"js8call",
		    //   "tag_id":18667,
		    //   "tag":"f33",
		    //   "version":"2.2.0",
		    //   "user":"bodhi"
		    // }

		    assert msg['tag'] == "f33"

		    pkgName = msg['name']
		    kojiBuildID = msg['build_id'].toString()
		    
		    build (
			job: 'eln-build-pipeline',
			wait: false,
			parameters:
			    [
			    string(name: 'KOJI_BUILD_ID', value: kojiBuildID),
			])
		}
		script {
		    currentBuild.description = "${pkgName}: ${kojiBuildID}"
		    
                }
            }
        }
    }
}
