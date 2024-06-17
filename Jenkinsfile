
pipeline {
    agent any
    environment {
        GIT_CREDENTIALS_ID = 'GIT_HUB_ID'
        GIT_REPO_URL = 'https://github.com/mpakeeru/jenkins-ui.git'
    }
    stages {
        stage('User Selection') {
            steps {
                script {
                    def htmlContent = '''
                    <!DOCTYPE html>
                    <html lang="en">
                    <head>
                        <meta charset="UTF-8">
                        <title>Select Items</title>
                        <style>
                            .container { display: flex; flex-direction: column; align-items: center; margin-bottom: 20px; }
                            .row { display: flex; justify-content: space-between; width: 80%; margin-bottom: 20px; }
                            select { width: 200px; height: 300px; }
                            button { width: 50px; height: 30px; margin: 5px; }
                        </style>
                    </head>
                    <body>
                        <div class="container">
                            <h2>Select Environment</h2>
                            <select id="environment">
                                <option value="QA">QA</option>
                                <option value="TEST">TEST</option>
                                <option value="PROD">PROD</option>
                            </select>
                        </div>
                        <div class="container">
                            <h2>Select Lex Items</h2>
                            <div class="row">
                                <select id="lexLeft" multiple>
                                    <option value="lex-bot-1">Lex Bot 1</option>
                                    <option value="lex-bot-2">Lex Bot 2</option>
                                    <option value="lex-bot-3">Lex Bot 3</option>
                                </select>
                                <div>
                                    <button onclick="moveItems('lexLeft', 'lexRight')">></button>
                                    <button onclick="moveItems('lexRight', 'lexLeft')"><</button>
                                </div>
                                <select id="lexRight" multiple></select>
                            </div>
                        </div>
                        <div class="container">
                            <h2>Select Functions</h2>
                            <div class="row">
                                <select id="functionsLeft" multiple>
                                    <option value="lambda-function-1">Lambda Function 1</option>
                                    <option value="lambda-function-2">Lambda Function 2</option>
                                </select>
                                <div>
                                    <button onclick="moveItems('functionsLeft', 'functionsRight')">></button>
                                    <button onclick="moveItems('functionsRight', 'functionsLeft')"><</button>
                                </div>
                                <select id="functionsRight" multiple></select>
                            </div>
                        </div>
                        <div class="container">
                            <h2>Select Layers</h2>
                            <div class="row">
                                <select id="layersLeft" multiple>
                                    <option value="layer-1">Layer 1</option>
                                    <option value="layer-2">Layer 2</option>
                                    <option value="layer-3">Layer 3</option>
                                </select>
                                <div>
                                    <button onclick="moveItems('layersLeft', 'layersRight')">></button>
                                    <button onclick="moveItems('layersRight', 'layersLeft')"><</button>
                                </div>
                                <select id="layersRight" multiple></select>
                            </div>
                        </div>
                        <div class="container">
                            <h2>Select Modules</h2>
                            <div class="row">
                                <select id="modulesLeft" multiple>
                                    <option value="contact-module-1">Contact Module 1</option>
                                    <option value="contact-module-2">Contact Module 2</option>
                                </select>
                                <div>
                                    <button onclick="moveItems('modulesLeft', 'modulesRight')">></button>
                                    <button onclick="moveItems('modulesRight', 'modulesLeft')"><</button>
                                </div>
                                <select id="modulesRight" multiple></select>
                            </div>
                        </div>
                        <div class="container">
                            <h2>Select Flows</h2>
                            <div class="row">
                                <select id="flowsLeft" multiple>
                                    <option value="contact-flow-1">Contact Flow 1</option>
                                    <option value="contact-flow-2">Contact Flow 2</option>
                                </select>
                                <div>
                                    <button onclick="moveItems('flowsLeft', 'flowsRight')">></button>
                                    <button onclick="moveItems('flowsRight', 'flowsLeft')"><</button>
                                </div>
                                <select id="flowsRight" multiple></select>
                            </div>
                        </div>
                        <button onclick="submitSelection()">Submit</button>
                        <script>
                            function moveItems(fromId, toId) {
                                const fromBox = document.getElementById(fromId);
                                const toBox = document.getElementById(toId);
                                [...fromBox.selectedOptions].forEach(option => {
                                    toBox.appendChild(option);
                                });
                            }
                            function submitSelection() {
                                const getSelectedItems = (id) => [...document.getElementById(id).options].map(option => option.value);
                                const jsonPayload = JSON.stringify({
                                    environment: document.getElementById('environment').value,
                                    lex: getSelectedItems('lexRight'),
                                    functions: getSelectedItems('functionsRight'),
                                    layers: getSelectedItems('layersRight'),
                                    modules: getSelectedItems('modulesRight'),
                                    flows: getSelectedItems('flowsRight')
                                });
                                alert(jsonPayload); // For users to copy and paste the JSON output
                            }
                        </script>
                    </body>
                    </html>
                    '''
                    echo htmlContent
                    input message: 'Open the console output, select items and paste the JSON output here:',
                          parameters: [text(name: 'SelectedItems', description: 'Paste the JSON output here')]
                }
            }
        }

        stage('Process Selections') {
            steps {
                script {
                    // Parse the JSON from the input
                    def selectedItems = readJSON text: params.SelectedItems
                    def env = selectedItems.environment
                    def jsonFileName = "Deploy_To_${env}.json"
                    
                    // Write JSON to file
                    writeFile file: jsonFileName, text: groovy.json.JsonOutput.toJson(selectedItems)
                }
            }
        }

        stage('Push to GitHub') {
            steps {
                script {
                    // Clone the repo
                    sh 'git clone ${GIT_REPO_URL} repo'
                    dir('repo') {
                        // Copy the generated JSON file to the repo
                        def env = readJSON file: "Deploy_To_${env}.json"
                        sh "cp ../Deploy_To_${env}.json ."

                        // Commit and push changes
                        sh 'git config user.email "you@example.com"'
                        sh 'git config user.name "Your Name"'
                        sh 'git add .'
                        sh 'git commit -m "Add deployment JSON for ${env} environment"'
                        withCredentials([string(credentialsId: GIT_CREDENTIALS_ID, variable: 'GITHUB_PAT')]) {
                            sh "git push https://$GITHUB_PAT@github.com/yourusername/your-repo.git"
                        }
                    }
                }
            }
        }
    }
}
   