# GIT AND GITHUB INTEGRATION WITH JENKINS
### Assumptions made:
* Jenkins have necessary permission to read write and execute tasks remotely

**Following softwares are installed**
* DOCKER
* GIT
* JENKINS
* LINUX OS ( We have used RedHat 8 here)

# Steps in GIT
1. Create a new folder
    1. mkdir autoGit (you can name anything)
    
2. Initialize the newly created foldes as local git repository
    1. git init
    
3. Add remote repository as origin (the repository we will create in Github Section)
    1. git add remote origin `remote git repository url`
    
4. Create a file and push to remote repo to check if everything is working fine.
    1. cat > index.html (Add some text and press `ctrl + d`)
    2. git add index.html
    3. git commit -m "Commit messages"
    4. git push origin master 
    
5. Create one new branch `dev1`


# Github
1. Create a new repository
2. We will add files here using our local repository made above
3. Add a github webhook that will send post request to jenkins if push events happens

# Docker (This part is done in Red Hat 8 or any linux machine) 
1. Create a new folder to keep files which will be used to reflect changes in the server
    1. mkdir /wangweb (you can replace wangweb with any name of your choice)
2. Create two servers
    1. testOS for testing environment which will run at port 81
        - docker run -d -t -i -p 8081:80 -v /wangweb:/usr/local/apache2/htdocs/ --name testos httpd
    2. prodos for production environment which will run at port 82
        - docker run -d -t -i -p 8082:80 -v /wangweb:/usr/local/apache2/htdocs/ --name prodos httpd
        
# Jenkins
1. Create a new job `testJob` (you can give any name of your choice)
    1. This job will keep track of the dev1 branch
    2. Any push happens in the dev1 branch this job will clone the files and copy it to /wangweb folder (which we have created          in docker section)
    
    **Steps**
    1. `New Item` -> Enter job name ->śelect `Free Style Project` -> `ok`
    2. In `Source code management option`
        1. Select Git and add github repository url
        2. In `Branch Specifier` write dev1
    3. In build triggers check `GitHub hook trigger for GITScm polling`
    4. In build section -> Select `Execute Shell`
        1. Inside Execute Shell Ente the following command
            - sudo cp -v -r -f /wangweb
    5. Save Changes
2. Create a new job `prodJob' 
    1. This job will keep track of the master branch.
    
    **Setup is same as testJob except**
    `Do not change anything in branch specifier (In testJob we wrote dev1 there). Let it be as it is.
3. Create a new job `qat`
    1. This job will automatically merge the dev1 branch with the master branch
    2. This will not build automatically
    3. Quality Assurance Team checks the server at testing environment (testos) and if the new changes is working fine there
    4. Build ( This will automatically build prodJob due to webhook feature (go to github step 3 for this)
    
    **Setup**
    Only change in configuration , Creation is same as the above two
    1. In `Source Code Management`
        1. Enter github repository url
        2. Add Credentials ( Just add github username and password, left other options blank)
    2. In `Additional Behaviour`
        1. Add `merge before build`
            1. Name of the repository - origin
            2. Branch to merge to - dev1
            3. Merge Strategy - octopus
            4. go to post build option in the last
    3. In `post-build` option
        1. Add Git Publisher
        2. Check `Push only if build succeeds`
        3. Check `Merge results`
    4. SAVE CHANGES

# We can use ngrok to test our integration
    ## It will be used to host our server publicly
