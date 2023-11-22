pipeline {
    agent {
        docker {
            image 'python:latest'
            args '-u root'
        }
    }
    stages {
        stage("test") {
        steps {
            sh 'python -m coverage run manage.py test'
        }
        }
    }
    post{
    	always{
    	    archiveArtifacts 'htmlcov/*'
    	    cobertura coberturaReportFile: 'coverage.xml'
    	}
    }
}
