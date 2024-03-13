# dive-elasticsearch-docker
Elasticsearch custom docker image for DIVE

#### v8.8.2
- Simplified further by deleting unused files and folders.
- Made changes to move the build from DIVE to DC Jenkins.
- Renamed dev to main branch and started using main
- Deleted master branch

#### v8.5.3
- In an effort to simplify custom image and eventually migrate to Elastic ELK images, simplified docker build. **config** folder(s) from GitHub are not used in the image.  Config files are mounted from K8 configmaps.  **cacerts.pem** is also not used any more.

#### v7.17.4
- **config** folder is DIVE docker-swarm specific ES configuration folder.
- **config-eks** folder is DIVE EKS specific ES configuration folder.
- **config-default** is generic ES configuration kept for reference when DIVE specific configuration (config folder) was moved inside the container during EKS migration.  Contents inside these folders are not in docker build.

