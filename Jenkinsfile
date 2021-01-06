pipeline {
	agent any	
	
	environment {
        BKP_FOLDER_DESTINATION = ''
        ROLLBACK = false
    }
	
	stages {
	    
		stage('Restore Packages And Build') {
			steps { 
                powershell '''
                
                    cd\\; 
                    dotnet restore;
                    cd C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\workspace\\CoalaSpace-Test-Jenkins-WebAPI; 
                    dotnet build;

                '''
			}
		}
		
		stage('Unit testing') {
			steps { 
                powershell ''' 
                
                    cd\\; 
                    cd C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\workspace\\CoalaSpace-Test-Jenkins-WebAPI; 
                    dotnet test --logger trx; 
                    if ($lastexitcode -gt 0) { exit 1; } ;

                '''
			}
		}
		
		stage('Backup Application From IIS Server') {
			steps { 
                  script{
                      
                    def now = new Date()
                    BKP_FOLDER_DESTINATION = 'BKP_API_Client_' + now.format("dd-MM-yyyy.HH-mm-ss.SSS")  
                    bkpFolderDestination="${BKP_FOLDER_DESTINATION}"
                    withEnv(["folderDest=${bkpFolderDestination}"]){
                        
                        powershell ''' 
                        
                             $folderName =  $env:folderDest
                             New-Item -ItemType Directory -Path "D:\\Backups_Jenkins\\APIs\\Client\\" -Name $folderName;
                             $folderDestination = "D:\\Backups_Jenkins\\APIs\\Client\\$($folderName)";
                             Copy-Item -Path "D:\\web\\api\\Client\\*" -Destination $folderDestination –Recurse
                             
                         '''
                    }
                }
                
			}
		}
		
		stage('Deploy') {
			steps { 
                powershell '''
                
                    cd\\; 
                    cd C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\workspace\\CoalaSpace-Test-Jenkins-WebAPI;  
                    stop-WebAppPool -Name api-client; 
                    start-Sleep -s 10; 
                    Remove-Item D:\\web\\api\\Client\\* -Recurse -Force; 
                    start-Sleep -s 5; 
                    dotnet restore;
                    dotnet publish .//Achei.Client.Services.API//Achei.Client.Services.API.csproj -c release -o D:\\web\\api\\Client\\; 
                    start-WebAppPool -Name api-client;  

                '''
			}
		}
		
		stage('Integration testing') {
			steps {
			    script {
                    try {
                        powershell '''
                        
                            cd\\; 
                            cd C:\\Users\\Android\\AppData\\Roaming\\npm\\node_modules\\newman\\bin; 
                            node newman run C:\\TestIntegrationPostmanJenkins\\Test_Client_LocalHost.postman_collection.json  -r cli,json,junit,html --insecure --disable-unicode --reporter-junit-export C:\\TestIntegrationPostmanJenkins\\newman;

                        '''
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
                    withEnv(["FolderOrigin=${bkpFolderOrigin}"]){
                        powershell '''
                            $folderName= $env:FolderOrigin
                            write-host $folderName
                        '''
                        
                         powershell ''' 
                                    
                            $folderorigin = "D:\\Backups_Jenkins\\APIs\\Client\\" + $env:FolderOrigin + ”\\*”;
                            write-host $folderorigin
                            stop-WebAppPool -Name api-client; 
                            start-Sleep -s 10; 
                            Remove-Item D:\\web\\api\\Client\\* -Recurse -Force; 
                            start-Sleep -s 5; 
                            Copy-Item –Path $folderorigin  -Destination "D:\\web\\api\\Client\\"  –Recurse –force
                            start-Sleep -s 5; 
                            start-WebAppPool -Name api-client;  

                         '''
                    }
                }
            }
        }
        
	}
}