# TP INTEGRATION CONTINU 

# Membres du groupe
- TIANI DE HAPPI ULRICH
- MAHREZ SAIDI
- MATHIS CRINCHON
- JOHANNA PULULU
- YASSINE BELKHAYAT

# Objectif du projet : mettre en place une intégration continue Jenkins sur un projet réel.
   
# Build du code

Voici ci-dessous, point par point, comment nous avons procédé pour faire le Build du code sur JenKins:

## Première méthode:

- Dans Jenkins, nous avons créé un *nouvel item*.
- Nous avons saisi un nom pour notre item puis en avons construit un *projet free-style*.
- Dans la section *gestion du code source*, nous avons selectionné *Git* et copié l'URL de notre repository GitHub et nous y avons éfalement renseigné nos idenfiants de connexion GitHub. Pour ce qui est de la branche, nous avons chosi la *master* par défaut. 
- Dans la section *Build Step* nous avons ajouté l'étape *Invoquer les cibles Maven de haut niveau*. Nous avons également marqué *package* comme cible Maven.
- Dans la section *action a la suite du build* nous avons chosi d'archiver les *target/*.**jar*.
- Sauvegarder puis lancer le build.

## Deuxième méthode:

- Dans Jenkins, nous avons créé un *nouvel item*.
- Nous avons saisi un nom pour notre item puis en avons construit un *pipeline*.
- Dans la section Pipeline nous avons selectionné *Pipeline script for SCM* pour la définition, *Git* pour SCM. Nous avons ensuite renseigné l'URL de notre repository ainsi que nos identifiants de connexion GitHub. Nous avons égalemeent chosi la branche master.
- Sauvegarder puis lancer le build.

# Exécution des tests unitaires

Dans la pipeline il faut rajouter les lignes suivantes: 

```
stage('test') {
            steps {
                bat 'mvn clean'
                bat 'mvn test' 
            }
        }
```


# Mise en place des tests statiques 
- Installation des plugins : Violations plugin & Warnings Next Generation Plugin
- Mise a jour de tous les plugins deja installés 
- Dans la configuration de l'item
  - Bloc "Post-build Actions" : Record compiler warnings and static analysis results
    - Tool : CheckStyle
  - save

# Génération des packages

Dans la pipeline il faut rajouter les lignes suivantes: 

```
stage('Package') {  
    steps {
            bat 'mvn clean'
            bat 'mvn package' 
        }
    }
```

# Publication du package sur Nexus

- Créer un repo nexus, aller dans "create repository"
- Selectionner "maven2 (hosted)"
- Le nommer "simple-astronomy-lib"
- Mettre "Version Policy" sur "Mixed"
- Mettre "Deployement Policy" sur " Allow - - - Redeploy" pour pouvoir redeployer l'application comme on le souhaite 
- Créer un user Jenkis en allant dans "Dashboard > Server Administrator and Configuration > User > Create user" avec les informations suivantes :

   - ID: jenkins-user
   - First Name: Jenkins
   - Last Name: User
   - Email: jenkins@jenkins.com
   - Password: Jenkins
   - Status: Active
   - Roles: nx-admin 
   
- Installer le Nexus Plugin sur Jenkins :
    
    - Aller dans "Tableau de bord > Administrer Jenkins > Gestion des Plugins > Disponible"
    - Chercher et Installer "Nexus Artifact Uploader"
    - Chercher et installer "Pipeline Utility Steps"

- Ajouter les identifiant de connexion pour Nexus sur Jenkins :
"Tableau de bord > Administrer Jenkins > Manage credentials > System > identifiant Globaux > add domain" : 

     - Nom du domaine : Nexus 
     - Nexus local credentials

- ensuite "Nexus> add credentials"

     - Nom d'utilisateur : jenkins-user
     - Mot de passe : Jenkins
     - ID : nexus-user-credential

Ensuite il suffit d'ajouter dans le pipeline jenkis le stage("Publish to Nexus Repository Manager") qui  est deja disponible dans notre JenkisFile.
Au début du JenkinsFile dans "environnement" il faut changer les identifiants de connexion à nexus : 

Pour notre TP ce sera ceux-ci: 
    
    NEXUS_VERSION = "nexus3"
    NEXUS_PROTOCOL = "http"
    NEXUS_URL = "localhost:8081/"
    NEXUS_REPOSITORY = "simple-astronomy-lib"
    NEXUS_CREDENTIAL_ID = "nexus-user-credential"

On a du enlever dans la pipeline le "post" qu'on a vu en cours car il ne fonctionnait pas et mettait notre exécution en échec :
```
post {
        always {
        junit '**/surefire-reports/*.xml'
        
        recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
        recordIssues enabledForFailure: true, tool: checkStyle()
        recordIssues enabledForFailure: true, tool: spotBugs()
        recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
        recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
        
    }
}
```    

 <font color="#ff726f"> Il faut cepandant autoriser les script pour que notre étape fonctionnes, pour cela aller dans "Tableau de bord > Administrer Jenkins > In-process Script Approval"
Rajouter la ligne suivante dans "Signatures already approved:" 
    method org.apache.maven.model.Model getPackaging.
</font>
