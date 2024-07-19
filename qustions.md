Tech 1: Git hub
1. branching strategy (develop for upcoming release, develop-next for future release, hotfix for bug fix, master  , then we have feature branches which merges in develop for release)












2. security : branches security, checks and approval 






3. git stash


git stash is a powerful command in Git that allows you to temporarily save changes in your working directory that you do not want to commit immediately. This can be particularly useful when you need to switch branches or pull updates from the remote repository without committing unfinished work.












4. git reset



it reset command is used to undo changes in your Git repository. It can be employed to undo commits, unstage changes, and modify the working directory in various ways. Understanding the different options available with git reset is essential to using it effectively. Here’s an in-depth explanation of git reset and its primary uses:

Types of git reset
Soft Reset (--soft)
Mixed Reset (--mixed)
Hard Reset (--hard)
Basic Usage
Soft Reset (--soft)

A soft reset moves the HEAD pointer to a specified commit but leaves your working directory and index (staging area) unchanged. This is useful when you want to undo a commit but keep the changes staged.

bash
Copy code
git reset --soft <commit>
Example:

bash
Copy code
git reset --soft HEAD~1
This moves the HEAD pointer back by one commit, leaving the changes from that commit staged.

Mixed Reset (--mixed)

A mixed reset (the default option) moves the HEAD pointer to a specified commit and resets the index to match that commit, but leaves your working directory unchanged. This unstages all changes that were staged for commit.

bash
Copy code
git reset --mixed <commit>
Example:

bash
Copy code
git reset --mixed HEAD~1
This moves the HEAD pointer back by one commit and unstages the changes from that commit.

Hard Reset (--hard)

A hard reset moves the HEAD pointer to a specified commit and resets both the index and working directory to match that commit. This discards all changes in the working directory and index.

bash
Copy code
git reset --hard <commit>
Example:

bash
Copy code
git reset --hard HEAD~1
This moves the HEAD pointer back by one commit and discards all changes from that commit in the working directory and index.

Practical Examples
Undo the Last Commit, Keep Changes Staged:

bash
Copy code
git reset --soft HEAD~1
This command undoes the last commit but keeps your changes in the staging area.

Unstage Changes:

bash
Copy code
git reset HEAD <file>
This unstages the specified file(s) but keeps the changes in the working directory.

Discard All Changes and Commits:

bash
Copy code
git reset --hard origin/main
This resets your local branch to match the remote main branch, discarding all local changes and commits.

Detailed Example Workflow
Make a Commit:

bash
Copy code
echo "Some changes" > file.txt
git add file.txt
git commit -m "Added some changes"
Undo the Last Commit (Keep Changes Staged):

bash
Copy code
git reset --soft HEAD~1
Unstage the Changes:

bash
Copy code
git reset HEAD file.txt
Discard the Changes in the Working Directory:

bash
Copy code
git checkout -- file.txt
Conclusion
The git reset command is a powerful tool for managing your commit history and working directory state. By understanding the differences between soft, mixed, and hard resets, you can effectively control your Git repository and recover from mistakes. Use it with care, especially the --hard option, as it can permanently discard changes.













Tech 2:     CI/CD : Continuous delivery and continuous Deployment
ADO : Azure Devops
This is CI part
1. having azure-pipeline.yml file in app code itself.
   check what is happening here:
   
   Above yml file has all the declaration/process mentioned for a build/compile as below:
         - which branch needs to trigger for a build
         - which branch should trigger a build against a PR
         - if any template to call that we can mention // optional
         - if any variable to call from ADO then we can mentioned those as variable in pipeline. These variable cann be declared in Library section of ADO.
         - Then build type : maven/gradle/NPM
         - create Dockerfile with build artifact and tag them as latest and unique build-ID (this build ID is hash from ADO build auto generated)
         - integrate sonarcloud check (we have sonar-properties in app code itself)


         - then push Docker file to JFROG artifactory
         - push helm chart to ADO workspace (we use helm chart package manager for kubernetes deployment)
           This helm chart contains resources which an application require to deploy on ocp cluster likewise: 
              Deployment, configmap, secret, service, route(ingress in opensource kubernetes) etc.
              
This is CD part:
2. Once build completes it creates a Release with unique timestamp and docker build tag with incremental numbers
3. Each deployment from develop branch has to go through below stages until it reaches to production.
   - Dev, QA, UAT, Perf then Production
   
   Environment explanation:
   Dev: Just for developer to examine their code if deployment has any issue.
   QA: QA runs the testing against all the feature being part of the upcoming release
   UAT: if require we do deploy the code on UAT for client to confirm the changes .
   PERF : Performance team runs their bench mark test which explains how an application is consuming the resources on machine like CPU, memory, disk space, Network I/o.
   Once we have all the approval from QA and Perf team then approved version is ready to deploy in production during the release night.
   
   - hotfix, production
   Any bug fix will be following the hotfix, production workflow. 
   
4. Each stage has all the task mentioned to deploy a build from ADO workspace and calling approved docker build tag. And tasks looks like this:
   - helm install on agent (we do run time on agent everytime and currently using version 3.0.0)
   - any cert/key or jks file download from library section if needed
   - deploy the helm chart using helm upgrade (This does uninstallation of old code and install new code/version of app)
   - for a testing side we do have `wait for deployment` tasks where pipeline wait until all the pods comes up in the mentioned stage.
   
   
   
   
Tech 3:  Docker
    - We use Docker just to create docker image using Dockerfile and push to the artifactory.
    
    Dockerfile contains basic info:
    Read : 
      - difference between RUN, CMD and Entrypoint

RUN:

Used during the image build process.
Executes commands and creates a new layer.
Permanent effect on the image



CMD:

Sets default commands/arguments for the container.

Can be overridden by docker run command arguments.

Only one CMD instruction is used; if multiple are provided, only the last one is applied.



ENTRYPOINT:

Sets the command to be executed when the container starts.
Cannot be overridden but can be appended with CMD or docker run arguments.
Preferred for defining the main behavior of the container.
Summary
RUN: Used to build the image, installing dependencies, and setting up the environment.
CMD: Provides default arguments for the container's main process.
ENTRYPOINT: Defines the main command that runs in the container.




  - how to inspect a docker container
      



 - how to run a docker container from an image using bash
      
      

Tech 4: Helm chart
    - helm create
    - helm upgrade
    - helm uninstall
    - CRDS skip for helm
    - passing argument for helm for runtime
    
    
Tech 5: Kubernetes
    - All our helm template yml file is in Appcode repo itself.
    - We have two container running in one pod (one for app and one for nginx)
        - why nginx ? since we have multiple apps using aliases for same code in backend we need nginx to redirect them to appropriate request when we hit the route url.
    - when a deployment happens it creates below for us:
         - Deploymentconfig
         - pods
         - configmap
         - secrets
         - service
         - route
         - horizontal pod Autoscaler (for few apps) : 
            https://github.com/KAR-AUTO/ol-auction-engine/blob/develop/deployment/helm/chart/templates/hpa-0.yml
    - We dont use any statefulset application like Database in current kubernetes setup. All DB machines are on vsphere and we connect them via DB credential from app.
    
    
Basic Issues/Questions with kubernetes we see:
1. pod crashloopbackoff issue:
   reason:
     1. image not available
     2. image naming convention
     3. readiness/liveness probes failure
     4. configmap/secret issues
     5. DB credential issue
     
2. How do we scale pods on demand:
     1. using min/max replica with CPU percentage usage by declaring in deployment file
     
3. Resource allocation using Limit memory and cpu:
   
4. What is port, targetport, nodeport in service.
5. what type of expose you use like ClusterIP, loadbalancer, ExternalIP, nodeport.






      we use ClusterIP
      
6. We expose service and creates route then from loadbalancer on network team side (using AVI LB tool) they configure either app needs to be outside or within VPN.
