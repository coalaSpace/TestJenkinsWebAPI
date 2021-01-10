pipeline {
	agent any	
	
	environment {
        BKP_FOLDER_DESTINATION = ''
        ROLLBACK = false
        DIRECTORY_WORKSPACE =  'C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\workspace\\CoalaSpace-Test-Jenkins-WebAPI'
        DIRECTORY_OF_BACKUP_APP_IIS_SERVER = 'D:\\Backups_Jenkins\\APIs\\Client\\'
        DIRECTORY_APP_IIS_SERVER = 'D:\\web\\api\\Client\\'
        PUBLISH_PROJECT = 'C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\workspace\\CoalaSpace-Test-Jenkins-WebAPI\\Achei.Client.Services.API\\Achei.Client.Services.API.csproj'
        TEST_POSTMAN_COLLECTION = 'C:\\TestIntegrationPostmanJenkins\\Test_Client_LocalHost.postman_collection.json'
        DIRECTORY_RESULT_TEST_POSTMAN = 'C:\\TestIntegrationPostmanJenkins\\newman\\API-Client'
        IIS_APP_POOL_NAME = 'api-client'
    }
	
	stages {
	    
	    stage('Checkout') {
			steps { 
                git branch: 'develop', url: 'https://github.com/coalaSpace/TestJenkinsWebAPI'
			}
		}
	    
		stage('Restore Packages And Build') {
			steps { 
                script{
                    directoryWorkSpace="${DIRECTORY_WORKSPACE}"
                    withEnv(["directoryWork=${directoryWorkSpace}"]){

                        powershell '''
                
                            cd\\;
                            cd $env:directoryWork; 
                            dotnet restore;
                            dotnet build;

                        '''
                    } 
                } 
			}
		}
		
		stage('Unit testing') {
			steps { 
                script{
                    directoryWorkSpace="${DIRECTORY_WORKSPACE}"
                    withEnv(["directoryWork=${DIRECTORY_WORKSPACE}"]){
                        powershell ''' 
                        
                            cd\\; 
                            cd $env:directoryWork; 
                            dotnet test --logger trx; 
                            if ($lastexitcode -gt 0) { exit 1; } ;

                        '''
                    }
                }
			}
		}
		
		stage('Backup Application From IIS Server') {
			steps { 
                  script{
                      
                    def now = new Date()
                    BKP_FOLDER_DESTINATION = 'BKP_API_Client_' + now.format("dd-MM-yyyy.HH-mm-ss.SSS")  
                    bkpFolderDestination="${BKP_FOLDER_DESTINATION}"
                    withEnv([ "folderDest=${bkpFolderDestination}", "backupFolder=${DIRECTORY_OF_BACKUP_APP_IIS_SERVER}", "dirAppIIS=${DIRECTORY_APP_IIS_SERVER}" ]){
                        
                        powershell ''' 
                        
                             $folderName =  $env:folderDest
                             $bkpFolder = $env:backupFolder
                             $appIIS = $env:dirAppIIS
                             New-Item -ItemType Directory -Path $bkpFolder -Name $folderName;
                             $folderDestination = "$($bkpFolder)$($folderName)";
                             Copy-Item -Path "$($appIIS)*" -Destination $folderDestination -Recurse -force
                             
                         '''
                    }

                    BKP_FOLDER_DESTINATION = "${DIRECTORY_OF_BACKUP_APP_IIS_SERVER}" + BKP_FOLDER_DESTINATION
                    echo "cesar Alexandre Akishigue Funaki"
                    echo BKP_FOLDER_DESTINATION
                }
			}
		}
		
	    stage('Deploy') {
			steps { 
                script{
                    withEnv([ "AppPoolName=${IIS_APP_POOL_NAME}", "pubProject=${PUBLISH_PROJECT}", "dirAppIIS=${DIRECTORY_APP_IIS_SERVER}"]){
                        powershell '''

                            $appIIS = $env:dirAppIIS
                            $pubProj = $env:pubProject
                            $poolName = $env:AppPoolName
                            
                            
                            stop-WebAppPool -Name $poolName; 
                            start-Sleep -s 10; 
                            Remove-Item"$($appIIS)*" -Recurse -Force; 
                            start-Sleep -s 5; 
                            dotnet restore;
                            dotnet publish $pubProj -c release -o "$($appIIS)"; 
                            start-WebAppPool -Name $poolName;   

                        '''
                    }
                } 
			}
		}
		
		stage('Integration testing') {
			steps {
			    script {
                    try {

                        withEnv([ "directoryNewman=${DIRECTORY_NEWMAN}", "postmanCollection=${TEST_POSTMAN_COLLECTION}", "dirResultNewman=${DIRECTORY_RESULT_TEST_POSTMAN}"]){

                            powershell '''
                            
                                $postmanCollect = $env:postmanCollection
                                $postmanResult = $env:dirResultNewman
                                cd\\; 
                                cd $env:directoryNewman; 
                                node newman run $postmanCollect --insecure --disable-unicode --reporter-junit-export $postmanResult;

                            '''
                            
                        }

                    } catch (Exception e) {
                        echo 'Entrou no Catch'
                        echo 'Teste de Integração falhou, o Rollback será executado'
                        echo 'Exception occurred: ' + e.toString()
                        ROLLBACK = true
                        currentBuild.result = 'FAILURE'
                        
                    }
                }
			}
		}
		
		stage('RollBack') {
		    agent any
            when{
              expression {
                    return ROLLBACK == true
                }
            }
            steps { 
                
                script{
                    bkpFolderOrigin="${BKP_FOLDER_DESTINATION}"
                    withEnv(["FolderOrigin=${bkpFolderOrigin}", "dirAppIIS=${DIRECTORY_APP_IIS_SERVER}", "AppPoolName=${IIS_APP_POOL_NAME}"]){
                          
                         powershell ''' 
                                    
                            $appIIS = $env:dirAppIIS        
                            $folderorigin = $env:FolderOrigin + ”\\*”;
                            $poolName = $env:AppPoolName

                            write-host $folderorigin

                            stop-WebAppPool -Name  $poolName; 
                            start-Sleep -s 10; 
                            Remove-Item"$($appIIS)*" -Recurse -Force; 
                            start-Sleep -s 5; 
                            Copy-Item –Path $folderorigin  -Destination "$($appIIS)*"  –Recurse –force
                            start-Sleep -s 5; 
                            start-WebAppPool -Name  $poolName;  

                         '''
                    }
                }
            }
        }
        
	}
}